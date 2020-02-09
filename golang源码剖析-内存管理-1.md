# golang源码剖析-内存管理-1

​                                        转载                                                                                                                                            [robertkun](https://me.csdn.net/robertkun)                    最后发布于2018-04-29 00:50:20                                        阅读数 1344                                                                                                                        收藏                                    

​                                                                展开                                    

转自[http://skoo.me/go/2013/10/13/go-memory-manage-system-alloc ] 
 这个拿来主义虽然不太好, 但总比不拿强.. 
 吃水不忘挖井人,感谢原文作者分享.

内存布局结构图 
 ![这里写图片描述](https://img-blog.csdn.net/20180429003820529?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3JvYmVydGt1bg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

我把整个核心代码的逻辑给抽象绘制出了这个内存布局图，它基本展示了Go语言内存分配器的整体结构以及部分细节（这结构图应该同样适用于tcmalloc）。从此结构图来看，内存分配器还是有一点小复杂的，但根据具体的逻辑层次可以拆成三个大模块——**cache，central，heap**，然后一个一个的模块分析下去，逻辑就显得特别清晰明了了。 
 位于结构图最下边的Cache就是cache模块部分； 
 central模块对应深蓝色部分的MCentral，central模块的逻辑结构很简单，所以结构图就没有详细的绘制了； 
 Heap是结构图中的核心结构，对应heap模块，也可以看出来central是直接被Heap管理起来的，属于Heap的子模块。

在分析内存分配器这部分源码的时候，首先需要明确的是所有内存分配的入口，有了入口就可以从这里作为起点一条线的看下去，不会有太大的障碍。这个入口就是/src/runtime/malloc.go源文件中的runtime·mallocgc函数，这个入口函数的主要工作就是分配内存以及触发gc(本文将只介绍内存分配)，在进入真正的分配内存之前，此入口函数还会判断请求的是小内存分配还是大内存分配(32k作为分界线)；小内存分配将调用mcache.alloc函数从Cache获取，而大内存分配调用mheap.alloc直接从Heap获取。入口函数过后，就会真正的进入到具体的内存分配过程中去了。

先上一大段的代码吧. 
 其中小内存分配走tiny,大内存分配走large…

```
// Allocate an object of size bytes.
// Small objects are allocated from the per-P cache's free lists.
// Large objects (> 32 kB) are allocated straight from the heap.
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
    if gcphase == _GCmarktermination {
        throw("mallocgc called with gcphase == _GCmarktermination")
    }

    if size == 0 {
        return unsafe.Pointer(&zerobase)
    }

    if debug.sbrk != 0 {
        align := uintptr(16)
        if typ != nil {
            align = uintptr(typ.align)
        }
        return persistentalloc(size, align, &memstats.other_sys)
    }

    // assistG is the G to charge for this allocation, or nil if
    // GC is not currently active.
    var assistG *g
    if gcBlackenEnabled != 0 {
        // Charge the current user G for this allocation.
        assistG = getg()
        if assistG.m.curg != nil {
            assistG = assistG.m.curg
        }
        // Charge the allocation against the G. We'll account
        // for internal fragmentation at the end of mallocgc.
        assistG.gcAssistBytes -= int64(size)

        if assistG.gcAssistBytes < 0 {
            // This G is in debt. Assist the GC to correct
            // this before allocating. This must happen
            // before disabling preemption.
            gcAssistAlloc(assistG)
        }
    }

    // Set mp.mallocing to keep from being preempted by GC.
    mp := acquirem()
    if mp.mallocing != 0 {
        throw("malloc deadlock")
    }
    if mp.gsignal == getg() {
        throw("malloc during signal")
    }
    mp.mallocing = 1

    shouldhelpgc := false
    dataSize := size
    c := gomcache()
    var x unsafe.Pointer
    noscan := typ == nil || typ.kind&kindNoPointers != 0
    if size <= maxSmallSize {
        if noscan && size < maxTinySize {
            // Tiny allocator.
            //
            // Tiny allocator combines several tiny allocation requests
            // into a single memory block. The resulting memory block
            // is freed when all subobjects are unreachable. The subobjects
            // must be noscan (don't have pointers), this ensures that
            // the amount of potentially wasted memory is bounded.
            //
            // Size of the memory block used for combining (maxTinySize) is tunable.
            // Current setting is 16 bytes, which relates to 2x worst case memory
            // wastage (when all but one subobjects are unreachable).
            // 8 bytes would result in no wastage at all, but provides less
            // opportunities for combining.
            // 32 bytes provides more opportunities for combining,
            // but can lead to 4x worst case wastage.
            // The best case winning is 8x regardless of block size.
            //
            // Objects obtained from tiny allocator must not be freed explicitly.
            // So when an object will be freed explicitly, we ensure that
            // its size >= maxTinySize.
            //
            // SetFinalizer has a special case for objects potentially coming
            // from tiny allocator, it such case it allows to set finalizers
            // for an inner byte of a memory block.
            //
            // The main targets of tiny allocator are small strings and
            // standalone escaping variables. On a json benchmark
            // the allocator reduces number of allocations by ~12% and
            // reduces heap size by ~20%.
            off := c.tinyoffset
            // Align tiny pointer for required (conservative) alignment.
            if size&7 == 0 {
                off = round(off, 8)
            } else if size&3 == 0 {
                off = round(off, 4)
            } else if size&1 == 0 {
                off = round(off, 2)
            }
            if off+size <= maxTinySize && c.tiny != 0 {
                // The object fits into existing tiny block.
                x = unsafe.Pointer(c.tiny + off)
                c.tinyoffset = off + size
                c.local_tinyallocs++
                mp.mallocing = 0
                releasem(mp)
                return x
            }
            // Allocate a new maxTinySize block.
            span := c.alloc[tinySpanClass]
            v := nextFreeFast(span)
            if v == 0 {
                v, _, shouldhelpgc = c.nextFree(tinySpanClass)
            }
            x = unsafe.Pointer(v)
            (*[2]uint64)(x)[0] = 0
            (*[2]uint64)(x)[1] = 0
            // See if we need to replace the existing tiny block with the new one
            // based on amount of remaining free space.
            if size < c.tinyoffset || c.tiny == 0 {
                c.tiny = uintptr(x)
                c.tinyoffset = size
            }
            size = maxTinySize
        } else {
            var sizeclass uint8
            if size <= smallSizeMax-8 {
                sizeclass = size_to_class8[(size+smallSizeDiv-1)/smallSizeDiv]
            } else {
                sizeclass = size_to_class128[(size-smallSizeMax+largeSizeDiv-1)/largeSizeDiv]
            }
            size = uintptr(class_to_size[sizeclass])
            spc := makeSpanClass(sizeclass, noscan)
            span := c.alloc[spc]
            v := nextFreeFast(span)
            if v == 0 {
                v, span, shouldhelpgc = c.nextFree(spc)
            }
            x = unsafe.Pointer(v)
            if needzero && span.needzero != 0 {
                memclrNoHeapPointers(unsafe.Pointer(v), size)
            }
        }
    } else {
        var s *mspan
        shouldhelpgc = true
        systemstack(func() {
            s = largeAlloc(size, needzero, noscan)
        })
        s.freeindex = 1
        s.allocCount = 1
        x = unsafe.Pointer(s.base())
        size = s.elemsize
    }

    var scanSize uintptr
    if !noscan {
        // If allocating a defer+arg block, now that we've picked a malloc size
        // large enough to hold everything, cut the "asked for" size down to
        // just the defer header, so that the GC bitmap will record the arg block
        // as containing nothing at all (as if it were unused space at the end of
        // a malloc block caused by size rounding).
        // The defer arg areas are scanned as part of scanstack.
        if typ == deferType {
            dataSize = unsafe.Sizeof(_defer{})
        }
        heapBitsSetType(uintptr(x), size, dataSize, typ)
        if dataSize > typ.size {
            // Array allocation. If there are any
            // pointers, GC has to scan to the last
            // element.
            if typ.ptrdata != 0 {
                scanSize = dataSize - typ.size + typ.ptrdata
            }
        } else {
            scanSize = typ.ptrdata
        }
        c.local_scan += scanSize
    }

    // Ensure that the stores above that initialize x to
    // type-safe memory and set the heap bits occur before
    // the caller can make x observable to the garbage
    // collector. Otherwise, on weakly ordered machines,
    // the garbage collector could follow a pointer to x,
    // but see uninitialized memory or stale heap bits.
    publicationBarrier()

    // Allocate black during GC.
    // All slots hold nil so no scanning is needed.
    // This may be racing with GC so do it atomically if there can be
    // a race marking the bit.
    if gcphase != _GCoff {
        gcmarknewobject(uintptr(x), size, scanSize)
    }

    if raceenabled {
        racemalloc(x, size)
    }

    if msanenabled {
        msanmalloc(x, size)
    }

    mp.mallocing = 0
    releasem(mp)

    if debug.allocfreetrace != 0 {
        tracealloc(x, size, typ)
    }

    if rate := MemProfileRate; rate > 0 {
        if size < uintptr(rate) && int32(size) < c.next_sample {
            c.next_sample -= int32(size)
        } else {
            mp := acquirem()
            profilealloc(mp, x, size)
            releasem(mp)
        }
    }

    if assistG != nil {
        // Account for internal fragmentation in the assist
        // debt now that we know it.
        assistG.gcAssistBytes -= int64(size - dataSize)
    }

    if shouldhelpgc {
        if t := (gcTrigger{kind: gcTriggerHeap}); t.test() {
            gcStart(gcBackgroundMode, t)
        }
    }

    return x
}123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232
```

ok, 今天就到这里, 5.1假期到了, 上半年的假的还是比较多的, 不过每个假期其实都很累啊…