# Go Vim Debugging with GDB

 		[2017/02/15](https://michaelthessel.com/go-vim-debugging-with-gdb/) [Michael Thessel](https://michaelthessel.com/author/admin/)[Debug](https://michaelthessel.com/tag/debug/), [GDB](https://michaelthessel.com/tag/gdb/), [Go](https://michaelthessel.com/tag/go/), [Golang](https://michaelthessel.com/tag/golang/)		

To improve my Go workflow I wanted to tie a Go code debugger into VIM.

Even though Go is a mature language there doesn’t seem to be an officially supported debugger at this point in time.

I found 3 different projects:

- GDB ([Go usage](https://golang.org/doc/gdb))
- [delve](https://github.com/derekparker/delve)
- [godebug](https://github.com/mailgun/godebug)



Godebug has been abandoned since September 2015. At this point Delve  seems to be the most promising debugger for Go. Delve has been  integrated into IntelliJ and Visual Studio Code. Unfortunately there is  no VIM integration available. There has been a [ticket](https://github.com/fatih/vim-go/issues/233) discussing integrating delve into vim-go but the ticket has been rejected.

The only viable option at the moment seems to be to integrate GDB  into VIM. GDB is the debugger listed in the official Go documentation.  The documentation also contains this caveat:

*GDB does not understand Go programs well. The stack management,  threading, and runtime contain aspects that differ enough from the  execution model GDB expects that they can confuse the debugger, even  when the program is compiled with gccgo. As a consequence, although GDB  can be useful in some situations, it is not a reliable debugger for Go  programs, particularly heavily concurrent ones. Moreover, it is not a  priority for the Go project to address these issues, which are  difficult. In short, the instructions below should be taken only as a  guide to how to use GDB when it works, not as a guarantee of success.*

I haven’t done any concurrent debugging tests with Go and GDB but I  could successfully debug any non-concurrent code with GDB. I tested this with a variety of projects and despite the discouraging message in the  documentation, GBD worked very stable and reliable during all my tests. I decided to go for GDB for now till there is a better option available.

To integrate GDB into VIM you need 2 plugins:

```
Plugin 'fatih/vim-go'   
Plugin 'vim-scripts/Conque-GDB'
```

Anybody who develops Go in VIM will be familiar with [vim-go](https://github.com/fatih/vim-go). It ties Go into VIM very nicely and is basically the default VIM plugin for Go developers. Vim-go also forms the basis for debugging Go with [Conque-GDB](https://github.com/vim-scripts/Conque-GDB). Conque-GDB is just a generic GDB plugin for VIM, but it works very well with Go.

After installing both plugins you need to add the following lines to your .vimrc

```
let g:ConqueTerm_Color = 2                                                            
let g:ConqueTerm_CloseOnEnd = 1                                                       
let g:ConqueTerm_StartMessages = 0                                                    
                                                                                      
function DebugSession()                                                               
    silent make -o vimgdb -gcflags "-N -l"                                            
    redraw!                                                                           
    if (filereadable("vimgdb"))                                                       
        ConqueGdb vimgdb                                                              
    else                                                                              
        echom "Couldn't find debug file"                                              
    endif                                                                             
endfunction                                                                           
function DebugSessionCleanup(term)                                                    
    if (filereadable("vimgdb"))                                                       
        let ds=delete("vimgdb")                                                       
    endif                                                                             
endfunction                                                                           
call conque_term#register_function("after_close", "DebugSessionCleanup")              
nmap <leader>d :call DebugSession()<CR>;   
```

With these settings in place you can start a GDB debug session simply by typing <leader>d.

This will

1. Compile your app with the recommended debug switches
2. Start a debug session with the generated binary
3. Clean up after you complete debugging

If you see the following error message when starting a debug session:

```
warning: File "/usr/local/go/src/runtime/runtime-gdb.py" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
To enable execution of this file add
add-auto-load-safe-path /usr/local/go/src/runtime/runtime-gdb.py
line to your configuration file "/home/example/.gdbinit"
```

Then simply create a file called *.gdbinit* in your home directory and add this line to it:

```
add-auto-load-safe-path /usr/local/go/src/runtime/runtime-gdb.py
```

If you have Go installed somewhere else you need to adjust the path accordingly.

Once the debug session started you have the following actions available in your buffer:

- <leader>r: Run
- <leader>c: Continue
- <leader>n: Execute next line
- <leader>p: Print variable under cursor
- <leader>b: Toggle breakpoint

You can switch between the debug window and your buffer using the  regular <ctrl>w movements. If you want to type commands directly  into the debug window you can switch there and enable insert mode. To  switch back to the buffer you need to exit insert mode first.

Apart from the provided Conque-GDB mappings the GDB command line offers quite a few additional features:

- p variablename: Print variable
- bt: Print backtrace
- info goroutines: Go routine list
- goroutine n bt: Print backtrace for a specific goroutine
- b main.go:5: Add a breakpoint to main.go on line 5
- d 8: Delete breakpoint number 8
- quit: End debug session

You can find the full list in the [Go GDB documentation](https://golang.org/doc/gdb).

Here some screenshots of an example debug session:
 [![img](http://michaelthessel.com/wp-content/uploads/2017/02/gogdb.gif)](http://michaelthessel.com/wp-content/uploads/2017/02/gogdb.gif)

**Edit:**

/u/KenjiTakahashi pointed out that there is the [vim-godebug plugin](https://github.com/jodosha/vim-godebug). Vim-godebug has Delve support. Unfortunately only works for NeoVim. At  the time of writing this plugin is only a week old. I assume there will  me more development in the future and maybe even Vim support.