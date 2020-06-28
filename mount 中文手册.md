# mount 中文手册

## 译者：**[金步国](http://www.jinbuguo.com/)**

------

### 版权声明

本文译者是一位开源理念的坚定支持者，所以本文虽然不是软件，但是遵照开源的精神发布。

- 无担保：本文译者不保证译文内容准确无误，亦不承担任何由于使用此文档所导致的损失。
- 自由使用：任何人都可以自由的阅读/链接/打印此文档，无需任何附加条件。
- 名誉权：任何人都可以自由的转载/引用/再创作此文档，但必须保留译者署名并注明出处。

### 其他作品

本文译者十分愿意与他人分享劳动成果，如果你对我的其他翻译作品或者技术文章有兴趣，可以在如下位置查看现有的作品集：

- [金步国作品集](http://www.jinbuguo.com/) [ http://www.jinbuguo.com/ ]

### 联系方式

由于译者水平有限，因此不能保证译文内容准确无误。如果你发现了译文中的错误(哪怕是错别字也好)，请来信指出，任何提高译文质量的建议我都将虚心接纳。

- Email(QQ)：70171448在QQ邮箱

------

```
MOUNT(8)                                    System Administration                                   MOUNT(8)


名称
       mount - 挂载文件系统


语法
       mount [-lhV]

       mount -a [-fFnrsvw] [-t vfstype] [-O optlist]

       mount [-fnrsvw] [-o option[,option]...]  device|dir

       mount [-fnrsvw] [-t vfstype] [-o options] device dir


描述
       此命令的标准格式是

              mount -t type device dir

       原 dir 里面的 内容/属主/权限 将被屏蔽，直到此设备被卸载。
       如果只给出了 dir 或者只给出了 device ，那么将根据 /etc/fstab 的设置进行挂载。
       如果想避免 dir 与 device 之间的混淆，可以使用 --target(表示dir) 或 --source(表示device) 进行明确标明：

              mount --target /mountpoint

       关于 device 的说明：
              大多数的 device 是一个类似 /dev/sda1 这样的块设备文件名，
              但是对于NFS来说则可能是 knuth.cwi.nl:/dir 的样子。
              此外还可以通过 LABEL(-L) 或 UUID(-U) 来标识一个块设备(uuid 中必须使用小写字母)。
              对于使用GPT分区格式的硬盘来说，还可以使用 PARTUUID 或 PARTLABEL 来标识。

              不要理所当然的认为 UUID 一定是唯一的，应该使用如下命令
              lsblk  -o +UUID,PARTUUID
              检查其是否真的在你的系统上是唯一的。

              推荐在 /etc/fstab 中使用 LABEL=label 来标记设备，
              而不是 /dev/disk/by-{label,uuid,partuuid,partlabel} 这样的符号链接。
              因为 LABEL 可读性更好，可移植性也更强。

              注意，mount 将 UUID 视为字符串(而不是16进制数)。UUID 中的字母必须小写。

              由于 proc 文件系统并不与特定的设备相关，所以挂载的时候通常使用"proc"来代替。

       /etc/fstab, /etc/mtab, /proc/mounts
              /etc/fstab(参见 fstab(5))中包含了描述 设备-挂载点-挂载选项 之间的对应关系。
              此文件的默认位置可以通过 --fstab 命令行选项进行修改(详见"命令行选项"小节)

              如下命令(通常用于启动脚本中)

                     mount -a [-t type] [-O optlist]

              将会挂载 /etc/fstab 中所有列出的所有设备(明确标记为"noauto"的除外)。
              如果额外加上 -F 选项，那么将同时并行挂载多个文件系统。

              如果挂载一个 fstab 或 mtab 中的文件系统，那么只要指定设备或挂载点之一就可以了。

              mount 和 umount 程序会在 /etc/mtab 中维护一份当前已挂载文件系统的列表。
              如果不传递任何参数，直接调用 mount 将会打印出此列表。

              如果同时给出了 device(或 LABEL, UUID, PARTUUID, PARTLABEL) 和 dir ，
              那么 mount 将不会读取 /etc/fstab 中的设置。例如：

                     mount /dev/foo /dir

              如果想在 /etc/fstab 已有选项的基础上追加选项(追加到已有选项之后)，可以使用下面的命令：

                     mount device|dir -o options

              当 proc 文件系统被挂载(假定挂载到 /proc)之后，/etc/mtab 和 /proc/mounts 中的内容差不多，
              但通常 /etc/mtab 中的内容更丰富一些(包含挂载选项)，但是更新可能不够及时(受 -n 选项的影响)。
              可以将 /etc/mtab 替换成指向 /proc/mounts 的符号链接，这有助于提升挂载/卸载大量文件系统的速度。
              当然，这样做会导致无法使用"user"选项，而且也会丢失一些信息。

       非root用户的挂载
              通常只有root用户可以挂载文件系统，
              但是如果在 fstab 中包含"user"选项，那么所有用户都可以挂载此文件系统。

              例如，如果在 fstab 中包含如下的行：

                     /dev/cdrom  /cd  iso9660  ro,user,noauto,unhide

              那么所有用户都可以通过下面的命令挂载 CDROM 上的 iso9660 文件系统：

                     mount /dev/cdrom
              或
                     mount /cd

              使用了"user"选项的文件系统，只有挂载该文件系统的用户才可以卸载它。
              若想允许任何其他用户卸载，那么可以使用"users"代替"user"来实现。

              "owner"选项与"user"类似，表示仅允许设备的属主挂载它。
              "group"选项与"owner"类似，表示仅允许设备的属组中的用户挂载它。

       绑定挂载
              可以将文件系统同时绑定到两个不同的位置：

                     mount --bind olddir newdir
              或者其短格式：
                     mount -B olddir newdir
              或者在 fstab 中添加如下的条目：
                     /olddir /newdir none bind

              这样，就可以从两个不同的位置访问完全相同的内容了。

              甚至可以将同一个文件或目录绑定到自身(只是创建了一个挂载点而已)：

                     mount --bind foo foo

              上面的绑定挂载只能绑定一个单独的文件系统，而不包含其下级子目录上的文件系统。
              如果想要递归绑定整个目录树上所有的文件系统，可以使用：

                     mount --rbind olddir newdir
              或者其短格式：
                     mount -R olddir newdir

              使用 --bind/--rbind 绑定挂载文件系统的时候，并不能改变其原有的挂载选项。
              如果想要改变挂载选项，必须在绑定之后，再使用 remount 选项来修改：

                     mount --bind olddir newdir
                     mount -o remount,ro newdir

              需要注意的是，remount 的行为受 /etc/mtab 的控制。
              第一个命令将'bind'标记记录到 /etc/mtab 文件，而第二个命令又会从 /etc/mtab 中读取到'bind'标记。
              如果你的系统上没有 /etc/mtab 文件，又或者你为 remount 明确的指定了 device 与 dir 两个参数，
              那么 /etc/mtab 将不会被读取，这就意味着你必须在此情况下明确使用 bind 标记：

                     mount --bind olddir newdir
                     mount -o remount,ro,bind olddir newdir

              注意，remount,ro,bind 将会创建一个只读的挂载点，但是原始文件系统的超级块依然是可写的。
              也就是说，olddir 是可写的，但是 newdir 却是只读的。

       移动操作
              可以通过下面的命令将一个目录移动到另一个地方，而保持文件的物理位置不变

                     mount --move olddir newdir
              或者其短格式：
                     mount -M olddir newdir

              这将导致 olddir 中的内容完全转移到 newdir 中来访问，但是文件的真实物理位置保持不变。
              注意：(1) olddir 必须是一个挂载点。(2) olddir 不能位于带有"shared"属性的挂载点之下(参见下文)。
              [提示]可以使用"findmnt -o TARGET,PROPAGATION /dir"命令查看挂载点 /dir 的属性。

       共享子树
              可以为一个挂载点(可以包含子挂载点)设置传播类型标记(shared, private, slave, unbindable)。
              shared 表示允许创建镜像，一个镜像内的挂载和卸载操作会被自动传播到所有其他镜像中。
              slave 表示自动继承主挂载点中挂载和卸载操作，但是自身的挂载和卸载操作不会反向传播到主挂载点中。
              private 表示既不继承主挂载点中挂载和卸载操作，自身的挂载和卸载操作也不会反向传播到主挂载点中。
              unbindable 表示禁止对该挂载点进行任何绑定(--bind|--rbind)操作。
              详见 Documentation/filesystems/sharedsubtree.txt 文档。

              支持的操作：
                     mount --make-shared mountpoint
                     mount --make-slave mountpoint
                     mount --make-private mountpoint
                     mount --make-unbindable mountpoint

              下面的命令表示递归的改变一个挂载点及其下的所有子挂载点的传播类型标记：
                     mount --make-rshared mountpoint
                     mount --make-rslave mountpoint
                     mount --make-rprivate mountpoint
                     mount --make-runbindable mountpoint

              [注意](1) 每个 mount 命令只能修改一个传播类型标记，也就是说不可以一次指定多个传播类型标记。
              (2) mount 在进行 --make-* 操作时不会读取 fstab(5) 文件，你必须在命令行上指定所有挂载选项。

              从 util-linux-2.23 版本开始，可以在 fstab(5) 中的"挂载选项"字段设置传播类型标记：
              (private, slave, shared, unbindable, rprivate, rslave, rshared, runbindable)


命令行选项
       完整的最终生效的选项由下列三部分组成：
       (1)首先，从 fstab 中得到的选项；
       (2)其次，命令行选项 -o 中设置的参数；
       (3)最后，再加上 -r 或 -w 选项(如果存在)。

       所有可用的命令行选项如下：

       -V, --version
              打印版本信息后退出

       -h, --help
              打印帮助信息后退出

       -v, --verbose
              啰嗦模式，显示处理过程的详细信息

       -a, --all
              挂载 /etc/fstab 中所有(符合指定类型的)的文件系统，但包含"noauto"标记的行除外。

       -F, --fork
              (与 -a 连用)为每个设备都产生一个新进程来挂载。这样可以并行地挂载不同的设备或连接不同的NFS服务器。
              这样做的好处是挂载速度更快；同时NFS服务器延时也是并行的。缺点是挂载顺序是不定的。
              因此，如果你想挂载 /usr 和 /usr/spool 的话就不能使用这个选项。

       -f, --fake
              除了不实际执行挂载的系统调用之外，其它都和不使用这个选项相同。
              通常和 -v 选项联用以查看挂载动作究竟做了些什么事情。
              也通常用于向 /etc/mtab 中添加先前被 -n 屏蔽掉的挂载信息。
              -f 选项会检查 /etc/mtab 中已有的条目，并在已有重复条目的情况下返回失败的结果。

       -i, --internal-only
              不调用 /sbin/mount.TYPE 辅助程序，即使它真实存在。

       -l, --show-labels
              在挂载后输出文件系统的卷标签。mount 必须有读取磁盘设备的权限。

       -n, --no-mtab
              不将挂载信息写入 /etc/mtab ，当此 /etc 位于只读文件系统上的时候，通常就需要使用它。

       -c, --no-canonicalize
              不对路径(path)进行规范化。
              mount 默认会将所有来自命令行和 fstab 的路径进行规范化之后再写入 /etc/mtab 文件。
              这个选项可以和 -f 一起用于已经规范化了的绝对路径。

       -s     忽略文件系统不支持的挂载选项而不是导致挂载失败。
              目前只有 mount.nfs 挂载帮助程序支持此选项。

       --source src
              如果只为 mount 给出了一个参数，为避免混淆(挂载点/设备)，
              可以使用此选项强制表示 src 是"设备"而不是"挂载点"。

       -r, --read-only
              以只读模式挂载。等价于"-o ro"。

              注意，指定此选项后内核仍然有可能向磁盘写入数据(取决于不同的文件系统)。
              例如 ext3/ext4 会在检测到文件系统未被正确卸载的情况下，重放日志。
              要避免这种行为，你可以在挂载 ext3/ext4 的时候，使用"ro,noload"选项，
              或者用 blockdev(8) 工具将相应的块设备设置为"只读"模式。

       -w, --rw, --read-write
              以读写模式挂载，这是默认值。等价于"-o rw"。

       -L, --label label
              挂载标签名为"label"的分区。此选项要求 /proc/partitions 文件必须存在。

       -U, --uuid uuid
              挂载UUID为"uuid"的分区。此选项要求 /proc/partitions 文件必须存在。

       -T, --fstab path
              明确指定 fstab 文件的位置(默认是 /etc/fstab )。
              如果 path 是一个目录，那么此目录中的文件将由 strverscmp(3) 工具进行排序，
              并且以"."开头或者不以 .fstab 结尾的文件都将被忽略。
              可以多次使用此选项，以指定多个 fstab 文件。
              此选项的主要目的是用于 initramfs 或 chroot 环境中。

              注意，mount 本身并不将 --fstab 选项传递给 /sbin/mount.TYPE 帮助程序。
              也就是说，挂载帮助程序本身并看不到这里设定的 fstab 文件。
              所由于无法验证是否具有"user"这样的权限，以不能用于普通用户(非root)的挂载。

       -t, --types vfstype
              指定要挂载的文件系统类型。可以使用逗号分割多种类型。当前可用的类型有(实际情况取决于内核的配置)：
              adfs, affs, autofs, cifs, coda, cramfs, debugfs, devpts, efs, ext2, ext3, ext4, hfs,
              hfsplus, hpfs, iso9660, jfs, minix, msdos, ncpfs, nfs, nfs4, ntfs, proc, qnx4, ramfs,
              reiserfs, romfs, squashfs, smbfs, sysv, tmpfs, ubifs, udf, ufs, umsdos, usbfs, vfat, xfs

              mount 与 umount 还支持通过'.subtype'后缀定义的文件系统"子类型"(subtype)，例如'fuse.sshfs'。

              对于大多数文件系统，只要执行一个简单的 mount(2) 系统调用即可，并不需要明确指定文件系统的类型。
              但少数文件系统(nfs, nfs4, cifs, smbfs, ncpfs)则需要 /sbin/mount.TYPE 程序的帮助。

              如果没有给出此选项或者参数为"auto"，mount 将会使用 blkid 库进行猜测。
              如果猜测失败，mount 将读取 /etc/filesystems 文件，并尝试其中所有未被标记为"nodev"的文件系统类型。
              如果 /etc/filesystems 不存在，则 mount 将继续读取 /proc/filesystems 并进行同样的尝试。
              另一方面，如果 /etc/filesystems 以只包含单个 * 的一行结束的话，mount 将继续读取 /proc/filesystems 文件。
              所有这些探测的文件都将使用"silent"选项进行挂载。

              "auto"类型在挂载移动设备时很有用，
              而 /etc/filesystems 文件在改变探测顺序时很有用(特别是使用了内核模块自动加载功能的情况下)。
              例如，在 msdos 之前先尝试 vfat，或者在 ext2 之前先尝试 ext3 之类。

              可以用逗号分隔的列表来指定多个类型。也可以前缀"no"指示不使用这些文件系统。
              这种做法对于选项 -a 十分有用。例如：

                     mount -a -t nomsdos,ext2

              将挂载 msdos 和 ext2 之外的所有文件系统。

       --target dir
              如果只为 mount 给出了一个参数，为避免混淆(挂载点/设备)，
              可以使用此选项强制表示 dir 是"挂载点"而不是"设备"。

       -O, --test-opts opts
              与 -a 联用以限制 -a 处理的文件系统的集合。下面的命令

                     mount -a -O no_netdev

              表示挂载所有文件系统，但在 /etc/fstab 的选项字段中指定了 _netdev 的文件系统除外。

              它与 -t 的区别在于每个选项都被精确匹配；在一个选项开头前缀 no 不会影响其余选项。

              选项 -t 和 -O 的效果是累积的，也就是说，命令

                     mount -a -t ext2 -O _netdev

              挂载所有指定了 _netdev 选项的 ext2 文件系统，而不是 ext2 或指定了 _netdev 选项文件系统。

       -o, --options opts
              opts 是逗号分割的选项列表。例如：

                     mount LABEL=mydisk -o noatime,nouser

              更多信息参见下面的"文件系统无关的挂载选项"和"文件系统特定的挂载选项"。

       -B, --bind
              将某个目录树绑定挂载到其它地方，这样就可以同时从两个地方进行访问。

       -R, --rbind
              将某个目录树绑定挂载到其它地方，并且其子目录如果是挂载点的话也递归的进行绑定。

       -M, --move
              将某个目录树移动到另外一个地方。


文件系统无关的挂载选项
       这部分选项的当前值可以通过 /proc/mounts 查看。而其中一部分的默认值由内核编译时的配置决定。

       这里的选项与文件系统无关(适用于所有类型的文件系统)，并且都不可用于"rootflags="内核引导参数。

       下面的选项仅能在 /etc/fstab 文件中使用：

       auto   可以使用 -a 选项挂载。这是默认值。
       noauto 不能被自动挂载，也就是 -a 选项不会挂载它。

       group  允许非root用户挂载，如果该用户所属组之一匹配设备的属组。
              该选项隐含了 nosuid 和 nodev 选项，除非你特意进行了修改(比如：group,dev,suid)。
       owner  允许非root用户挂载，如果该用户是此设备文件的宿主的话。
              该选项隐含了 nosuid 和 nodev 选项，除非你特意进行了修改(比如：owner,dev,suid)。
       user   允许非root用户挂载此文件系统，此用户的名字将记入 mtab 中以便于随后再卸载。
              该选项隐含了 noexec, nosuid, nodev 选项，除非你特意进行了修改(比如：user,exec,dev,suid)。
       nouser 禁止非root用户挂载，这是默认值。
       users  允许所有用户挂载此文件系统。
              该选项隐含了 noexec, nosuid, nodev 选项，除非你特意进行了修改(比如：users,exec,dev,suid)。

       下面的选项既可以在 /etc/fstab 文件中使用，也可以在命令行中使用：

       async  所有的I/O操作都异步进行。这是默认值。
       sync   所有的I/O操作都同步进行(仅对 ext2, ext3, fat, vfat, ufs 有意义)。
              对于U盘/SSD之类的闪存设备，此选项可能缩短其寿命。
       dirsync
              所有对目录的更新操作都同步进行。这将影响下列系统调用：
              creat, link, unlink, symlink, mkdir, rmdir, mknod, rename

       atime  每一次访问文件与目录都更新inode访问时间(access time)
       noatime
              禁止更新文件与目录的inode访问时间，以获得更快的访问速度。
       diratime
              每一次访问目录都更新inode访问时间
       nodiratime
              禁止更新目录的inode访问时间，以获得更快的访问速度。

       iversion
              在每次修改inode的同时递增一次索引节点inode结构中"i_version"字段的值。
       noiversion
              禁止更新索引节点inode结构中"i_version"字段(被文件系统用来记录索引节点的改变)的值。

       relatime
              按照文件被修改的时间更改inode访问时间。也就是仅在文件的修改时间比访问时间新时才更新访问时间。
              与 noatime 类似，但是可以让 mutt 之类需要知道文件在最后一次被修改后是否被访问过的程序正常工作。
              从 Linux 2.6.30 起，这是默认值(除非指定了 noatime)。
              从 Linux 2.6.30 起，如果文件的最后访问时间已超过24小时未更新，也会被强制更新。
       norelatime
              禁止使用 relatime 特性，参见 strictatime 选项。

       strictatime
              允许明确的指定更新inode访问时间的方式，
              这样内核就可以在默认使用"relatime"或"noatime"的同时又允许用户空间的程序进行更改。
              更多默认的系统挂载选项可参见 /proc/mounts
       nostrictatime
              使用内核的默认行为更新inode访问时间

       lazytime
              仅在内存中更新各种时间戳(atime, mtime, ctime)而不真正刷写到磁盘上，此选项可以显著的提升高频随机写性能。
              仅在如下条件发生时，才会真正将时间戳刷写到磁盘上：
                  (1)该文件的inode因为非时间戳原因而需要更新
                  (2)应用程序明确的调用了 fsync(2), syncfs(2), sync(2) 函数
                  (3)一个未删除的 inode 被从内存中清理出去
                  (4)距离该 inode 上次刷写到磁盘的时间超过了24小时
       nolazytime
              禁用"lazytime"特性

       context=context
       fscontext=context
       defcontext=context
       rootcontext=context
              context= 选项用于挂载不支持扩展属性的文件系统，比如 VFAT 或在非SELinux系统下格式化的 ext3 分区。
              context= 还可以用于挂载你不信任的文件系统，比如软盘和U盘等可移动存储(软盘/U盘/光盘/移动硬盘/闪存卡)。
              例如，最常见用法的是：context="system_u:object_r:removable_t"

              另外两个选项 fscontext= 和 defcontext= 都与 context= 选项互斥。
              也就是说，你可以同时使用 fscontext= 和 defcontext= 选项，但是它们都不能和 context= 选项一起使用。

              fscontext= 可以用于所有文件系统，无论其是否有 xattr 支持。
              fscontext= 选项将超越不同架构的文件系统标签设置为一个特定的安全内容(security context)。
              此文件系统标签与单独的文件标签之间各自独立，
              它提供了某些针对整个文件系统范围的权限检查(例如在挂载时或创建文件时)。
              单独的文件标签依然可以从文件自身的xattr中获取。
              除了为每个文件提供相同的标签之外，context= 选项实际上设置了 fscontext= 选项提供的内容的集合。

              你可以使用 defcontext= 选项为不可标记的文件设置默认的安全内容(security context)。
              这将会覆盖策略中(policy)为不可标记的文件设置的值，前提是文件系统必须支持 xattr 标签。

              rootcontext= 选项允许你在文件系统被挂载之前就先对根inode进行明确的标记。
              这对于诸如Stateless Linux这样的无状态系统很有用。

              注意，内核会拒绝任何包含这四个选项的 remount 请求，即使这些请求并未对内容(context)做出任何改变。

              警告：当 context 的值中包含逗号(,)时，必须要用引号进行界定，否则会导致错误或难以预料的结果。例如：

                  mount -t tmpfs none /mnt -o'context="system_u:object_r:tmp_t:s0:c127,c456",noexec'

              更多详情，请参阅 selinux(8) 手册页。


       ro     以只读模式挂载
       rw     以读写模式挂载，这是默认值。

       defaults
              等价于使用如下默认选项：rw, suid,  dev,  exec,  auto, nouser, async

       mand   允许强制锁定(mandatory locks)此文件系统，参见fcntl(2)。
       nomand 禁止强制锁定(mandatory locks)此文件系统，参见fcntl(2)。
              有两种对文件加锁的方式：劝告锁(advisory lock)和强制锁(mandatory lock)
              劝告锁没有强制性，内核只提供加锁以及检测文件是否已经加锁的手段，
              但是内核并不参与锁的控制和协调。如果有进程不遵守“游戏规则”，
              不检查目标文件是否已经由别的进程加了锁就往其中写入数据，那么内核是不会加以阻拦的。
              强制锁是一种内核强制采用的文件锁，每当有系统调用 open()、read()、write() 发生的时候，
              内核都要检查并确保这些系统调用不会违反在所访问文件上加的强制锁约束。
              也就是说，如果有进程不遵守游戏规则，硬要往加了锁的文件中写入内容，内核就会加以阻拦。

       dev    允许使用其中的字符设备和块设备文件。
       nodev  禁止使用其中的字符设备和块设备文件。

       exec   允许直接执行其中的二进制文件。
       noexec 禁止直接执行其中的二进制文件。

       suid   允许SUID和SGID位生效。
       nosuid 禁止SUID或SGID位生效。此选项表面上安全，但是如果安装了 suidperl(1) 就不安全了。

       _netdev
              表明此文件系统位于网络上。用于防止网络未启用的时候就挂载。

       nofail 即使指定的设备不存在也不报错

       silent 打开 silent 标记
       loud   关闭 silent 标记

       remount
              重新挂载一个已经挂载了的文件系统而不修改其挂载点。通常用于更改挂载选项(比如从 ro 变为 rw)。

              remount 对于命令行选项和 fstab 中选项的处理方式和 mount 完全相同，
              仅在 device 和 dir 都明确指定的情况下，fstab 与 mtab 才会被忽略。

              mount -o remount,rw /dev/foo /dir

              这个命令表示所有原来的挂载选项都将被替换，并且忽略 fstab 中预设的选项(除了loop=)。

              mount -o remount,rw  /dir

              这个命令表示将 fstab(或 mtab)中预设的选项与此处的命令行选项合并后得到新的挂载选项。

       x-*    所有以"x-"为前缀的选项都被看做注释或者特定于用户空间程序的选项。
              这些选项不会被记入 mtab 文件、发送给 mount.TYPE 帮助程序、mount(2) 系统调用。
              建议的格式是 x-APP.OPTION (例如 x-systemd.automount)

       x-mount.mkdir[=MODE]
              允许创建挂载点，MODE 用于指定访问权限，其默认值是"0755"。此选项仅允许root用户使用。


文件系统特定的挂载选项
       下面这些 -o 选项的标志仅可以用于特定的文件系统。

       注意，只有文件系统特定的挂载选项才可以用于"rootflags="内核引导参数中
       更多信息可以查看 Documentation/filesystems/ 目录


devpts
       devpts 是一个伪文件系统，一般挂载到 /dev/pts 目录，用于获取伪终端。
       进程打开 /dev/ptmx 所获取的为终端(slave)将可以通过 /dev/pts/NUM 访问。

       uid=value 与 gid=value
              为新创建的 PTY 指定 UID 和 GID 。如果未指定，那么将被设置为创建此设备的进程的 UID 和 GID 。

       mode=value
              为新创建的 PTY 设置权限模式。默认值是"0600"。

       newinstance
              创建一个单独的 devpts 文件系统实例。

              所有未使用此选项挂载的 devpts 实例之间共享同一个命名空间(pty编号共享)。
              而每个使用此选项挂载的 devpts 实例之间命名空间各自独立，也就是编号完全相同的pty之间也互不相干。

              此选项主要用于支持内核中的容器(container)功能。
              此选项仅在内核开启了 CONFIG_DEVPTS_MULTIPLE_INSTANCES 的时候才可用。

              为了更有效的使用此选项，/dev/ptmx 必须是一个连接到 pts/ptmx 的符号链接。
              详见 Documentation/filesystems/devpts.txt 文档。

       ptmxmode=value
              为新创建的 ptmx 设备节点设置默认的权限模式。默认值是"0000"(出于和旧内核的兼容性考量)。

              与 newinstance 联用可让每个实例在 devpts 文件系统的根(通常是 /dev/pts/ptmx)下都有专用的 ptmx 节点。

              此选项仅在内核开启了 CONFIG_DEVPTS_MULTIPLE_INSTANCES 的时候才可用。


proc
       uid=value 和 gid=value
              可以接受这两个选项，但是似乎并没有什么效果。


tmpfs
       Glibc 要求必须在 /dev/shm 上挂载 tmpfs 以满足POSIX共享内存的需要。

       size=字节数|百分比
              设置最大尺寸。默认的最大值是内存总量的一半。"size=0"表示没有限制。
              还可以使用百分数表示使用物理内存总数的比例。比如：size=50%

       nr_blocks=
              和 size= 一样，但是通过块(PAGE_CACHE_SIZE)的数量来设置。nr_blocks=0 表示没有限制。

       nr_inodes=
              设置最大inode数量。
              默认值：物理内存页数量的一半、所有lowmem内存页，两者中较小者。

       上面三个参数接受 k, m, g 作为后缀的尺寸(KB, MB, GB)。并且可以通过 remount 修改。

       uid=   设置挂载点根的初始UID。不可以通过 remount 修改。
       gid=   设置挂载点根的初始GID。不可以通过 remount 修改。
       mode=  设置挂载点根的初始权限。不可以通过 remount 修改。

       mpol=[default|prefer:Node|bind:NodeList|interleave|interleave:NodeList]
              为此实例中的所有文件设置NUMA内存分配策略，仅在内核支持NUMA(CONFIG_NUMA)时才可以使用。
              此选项可以在运行中通过"mount -o remount ..."进行调整。

              default
                     优先从本地节点分配内存

              prefer:Node
                     优先从给定的 Node 节点分配内存

              bind:NodeList
                     仅为 NodeList 列表中的节点分配内存

              interleave
                     优先以循环轮转方式在每个节点间分配内存

              interleave:NodeList
                     优先以循环轮转方式在 NodeList 指定的节点间分配内存

              NodeList 的格式是逗号分隔的十进制数列表及用连字符表示的区间。例如：mpol=bind:0-3,5,7,9-15

              如果内核不支持NUMA，或者 NodeList 中存在离线节点，此选项都会失败。


ext2
       这是最传统的Linux标准文件系统。
       大多数选项的默认值都由文件系统的超级块(superblock)决定，并可以使用 tune2fs(8) 工具调整。

       acl|noacl
              是否支持 POSIX Access Control Lists(需要内核开启CONFIG_EXT2_FS_POSIX_ACL)

       bsddf|minixdf
              设置系统调用 statfs 的行为。
              minixdf 表示在"f_blocks"字段返回文件系统的总块数，
              bsddf(默认值)则表示要减去被 ext2 文件系统所用而无法再存储文件的块数。
              因此会出现如下的差异：

              % mount /k -o minixdf; df /k; umount /k

              Filesystem  1024-blocks   Used  Available  Capacity  Mounted on
              /dev/sda6     2630655    86954   2412169      3%     /k

              % mount /k -o bsddf; df /k; umount /k

              Filesystem  1024-blocks  Used  Available  Capacity  Mounted on
              /dev/sda6     2543714      13   2412169      0%     /k

       debug  在每次 (re)mount 的时候输出调试信息。

       errors={continue|remount-ro|panic}
              定义遇到错误时的行为：
              continue 表示忽略错误，只将文件系统标记为不正确的，然后继续；
              remount-ro 重新只读挂载它；
              panic 则挂起系统。

       grpid|nogrpid
              定义新创建的文件应该获得怎样的GID。默认值是"nogrpid"。
              grpid 表示使用其父目录的GID；
              nogrpid 的含义如下：
                     (1)如果新建文件的父目录没有 setgid 位，那么使用创建此文件的进程的GID。
                     (2)如果新建文件的父目录带有 setgid 位，那么使用其父目录的GID；
                     (3)并且如果新建的是目录时，还要同样设置 setgid 位。

       nouid32
              禁用 32-bit UID/GID 以保持与旧版内核互操作

       oldalloc|orlov
              oldalloc 表示使用旧的块分配算法，orlov(默认)表示使用新的更快速的Orlov块分配算法。

       resgid=n
       resuid=n
              ext2会保留一定比例的可用空间(默认5%，参见mke2fs(8)和tune2fs(8))。
              这些选项决定了谁(拥有指定UID或者属于指定GID组的用户)可以使用这些保留的空间。

       sb=n   使用第 n 块而不是第一块作为超级块。在文件系统被损坏时，这样很有用。
              以前，超块在每8192个块处都会复制一个：1, 8193, 16385 ... 如果文件系统很大，将被复制很多次。
              后来 mke2fs 便默认使用 -s(稀疏超块)选项，可以减少超块备份的数量。
              这样做意味着使用较新的 mke2fs 创建的ext2文件系统无法在 Linux-2.0.* 中以读写方式挂载。
              这里块编号的单位是1k。因此，如果想使用以4k为单位的文件系统中的第32768块，应当用"sb=131072"。

       user_xattr|nouser_xattr
              是否支持"user."扩展属性(需要内核开启CONFIG_EXT2_FS_XATTR)

       xip    如果可能尽量使用就地执行(execute in place)特性


ext3
       ext3是ext2的日志加强版，除了上述ext2选项外，它还支持以下额外的选项

       journal=update
              更新 ext3 文件系统的日志为当前的格式。

       journal=inum
              如果一个日志已存在，这个选项将被忽略。否则，将用指定编号的inode保存日志文件，
              ext3将创建一个新日志，覆盖inode编号为inum的文件的原有内容。

       journal_dev=devnum/journal_path=path
              当外部日志设备的主/次设备号变化的时候，这个选项可以让用户重新指定日志的存储位置。
              这个位置既可以是"XXxx"十六进制形式表示的设备，也可以是 path 指定的设备节点文件。

       norecovery/noload
              在挂载时不读取 ext3 文件系统的日志。用于挂载已经损坏的文件系统。
              但是跳过文件系统日志重放，可能会导致各种错误。

       data={journal|ordered|writeback}
              指定文件数据的日志模式。元数据(metadata)总是被记入日志。
              要在根文件系统上指定日志模式，那么就必须使用例如"rootflags=data=journal"这样的内核引导参数

              journal
                     在写入文件系统之前，所有数据都首先被提交到日志中。这是最安全、性能最低的模式。

              ordered
                     这是默认的模式，所有数据在它的元数据被提交给日志之前，被强制直接写入文件系统。

              writeback
                     写入顺序不定，数据可能在元数据已被提交给日志之后写入文件系统。这是效率最高的方式。
                     它保证了文件系统内部的一致性，但是在崩溃和恢复后文件内可能出现旧数据。

       data_err={ignore|abort}
                     在 ordered 日志模式下，遇到文件内容的数据出错时如何处理。
                     默认值 ignore 忽略错误，但是打印出错误消息，而 abort 则直接退出。

       barrier=0 / barrier=1
              开启/关闭barrier特性。barrier=0 关闭，barrier=1 开启。ext3 默认关闭此特性。
              磁盘上配有内部缓存，以便重新调整批量数据的写操作顺序，优化写入性能，
              因此文件系统必须在日志数据写入磁盘之后才能写 commit 记录，
              若 commit 记录写入在先，而日志有可能损坏，那么就会影响数据完整性。
              barrier 强制日志以正确的次序提交到磁盘，这样就可安全的使用磁盘上的内部缓存，代价是降低一些性能。
              除非你的硬盘有后备电池之类的装载，请务必开启 barrier 特性，否则意外掉电可能会造成数据错误。

       commit=nrsec
              每 nrsec 秒向磁盘上刷新一次数据和 metadata ，默认值是5秒，0表示默认值。
              这样，即使忽然掉电，你也最多只损失最近 nrsec 秒的数据。较大的 nrsec 值带来更好的性能。
              5 秒或者更小的数字对性能有显著的不利影响，但是能让损失降到最低。


vfat
       uid=value
       gid=value
       umask=value
              设置该文件系统上所有文件的 uid/gid/umask
              默认值是当前进程的 uid/gid/umask
              umask 表示新建文件或目录的权限掩码

       allow_utime=value
              这个选项控制着对mtime/atime的权限检查
              20     如果当前进程和文件的GID相同，就可以更改时间戳
              2      所有用户都可以更改时间戳
              默认值从dmask选项中获得(如果目录可写，utime(2)也是被允许的，也就是 ~dmask & 022)

              utime(2)会检查当前进程是文件的属主还是拥有CAP_FOWNER特权，但是FAT文件系统本身并没有uid/gid信息，
              所以严格的权限检查根本行不通。通过这个选项，可以实现一些控制。

       codepage=value
              在FAT系列文件系统上，"8.3"格式的短文件名以特定的代码页进行存储(可以通过chcp命令查看)，
              但长文件名却以Unicode进行存储。此选项的作用就是指定将长文件名转换为短文件名时使用的代码页。
              默认使用内核的FAT_DEFAULT_CODEPAGE值。简体中文一般使用936代码页(codepage=936)。

       iocharset=value
              指定默认以什么字符集显示文件名，必须与系统的locale设置保持一致。默认使用内核的FAT_DEFAULT_IOCHARSET值。
              例如在"en_US.UTF-8"的情况下应该使用"utf8"。
              [注意]应谨慎使用"iocharset=utf8"，它会导致FAT文件系统上的文件名变得大小写敏感。

       utf8   UTF8是可以在终端(console)上使用的文件系统安全的Unicode编码。它可以在支持它的文件系统上启用。
              utf8=0, utf8=no, utf8=false 表示禁用。设置了'uni_xlate'后，UTF8就自动被禁止。

       uni_xlate
              将原始的Unicode字符转换为特殊的转义序列。这样允许你保存和恢复含有任意Unicode字符的文件名。
              没有这个选项的话，对于无法转换的名称将使用'?'代替。
              转义字符是':'，因为它在VFAT文件系统中是无效字符。

       nonumtail
              首先试着创建不带序列号码的短名称，然后再尝试 name~num.ext
              比如"longfilename.txt"将首先尝试"longfile.txt"是否冲突，然后再尝试"longfi~1.txt"

       usefree
              直接使用存储在FSINFO中的"可用簇数(free clusters)"值，而不是通过扫描磁盘来获得该值。
              因为Windows并不总是更新此值，所以仅在你确定FSINFO中的信息可靠的情况下才能使用此选项。

       quiet  打开quiet标志。即使某些操作失败也不显示错误信息。小心使用！

       check=value
              可以选择三种不同的文件名文件名大小写检查级别：
              s[trict]   大小写敏感
              r[elaxed]  忽略大小写
              n[ormal]   默认值。当前等价于r[elaxed]

       shortname={lower|win95|winnt|mixed}
              定义创建和显示满足8.3格式的短文件名时的行为。如果存在对应的长名字，那么总是显示长名字。
              有四种模式：
              lower  强制短名称在显示时转换为小写；当短名称不都是大写时保存一个长名称。
              win95  强制短名称在显示时转换为大写；当短名称不都是大写时保存一个长名称。
              winnt  原样显示短名称；当短名称不都是小写，也不都是大写时保存一个长名称。(推荐)
              mixed  原样显示短名称；当短名称不都是大写时保存一个长名称。这是默认值。

       tz=UTC 禁止将文件系统的时间戳在本地时间(Windows)和UTC时间(Linux)之间进行转换。
              在挂载原本就使用UTC时间戳的FAT设备(例如，数码相机存储卡)时该选项就很有用处了。

       showexec
              如果设置了此标记，那么仅当文件名后缀是EXE,COM,BAT之一时，才拥有执行权限。
              默认不设置此标记，所有文件都拥有执行权限。

       debug  打开debug标记，将输出文件系统的版本信息和参数列表(如果参数不一致，也会输出这些数据)。

       sys_immutable
              如果设置了此标记，FAT上的ATTR_SYS(系统文件)属性将被作为Linux上的IMMUTABLE(只读)标记处理。
              默认不设置此标记。

       flush  如果设置了此标记，文件系统将比通常情况下更早的刷新缓存。默认不设置此标记。

       rodir  在Windows上，目录的ATTR_RO(只读)属性已经没有实际意义。
              设置此标记可以让目录的ATTR_RO属性生效。

       errors=panic|continue|remount-ro
              定义遇到错误时的行为：continue忽略错误，只将文件系统标记为不正确的，然后继续；
              remount-ro重新只读挂载它；panic则挂起系统。默认值是remount-ro


ntfs
       nls=name
              返回文件名时使用的字符集。与VFAT不同，NTFS不允许文件名中包含无法转换的字符。

       uid=value, gid=value, umask=value
              设置文件系统中文件的权限。umask值以八进制值给出。
              默认情况下，文件所有者是root，不能被其他人读取。


ntfs-3g

    uid=value/gid=value
    设置文件和目录的UID和GID，默认值是当前进程的UID和GID。

    umask=value
    设置文件/目录的umask(8进制)，默认值是'000'，表示对所有用户都不屏蔽任何权限。

    usermapping=file-name
    使用file-name文件代替 .NTFS-3G/UserMapping 作为用户映射文件。
        如果file-name是绝对路径，那么该文件必须已经存在于一个先前已挂载的文件系统上。
        如果file-name是相对路径，那么该文件将被解释为相对于即将挂载的NTFS分区的目录。
        定义了用户映射文件之后，uid=, gid=, umask=, fmask=, dmask=, dsilent= 选项将被忽略。

    inherit
        当创建新文件时，根据父目录中定义的继承规则设置其属主和权限，
        这个规则违反了Posix标准，但是与Windows标准一致。
        用户映射文件的存在时此选项才有意义。

    ro  以只读模式挂载。挂载windows休眠状态下的磁盘时很有用处。

    remove_hiberfile
    与只读挂载不同，对于处于休眠状态的NTFS分区是禁止读写挂载的。
        要么唤醒windows并正常关闭，要么使用这个选项来移除windows休眠文件。
        但是请注意，使用此选项后已保存的Windows会话将会被全部移除。小心使用！

    recover, norecover
    在可能的情况下恢复或修复不一致或损坏的NTFS卷，默认值是recover

    atime    选项会在每次访问文件时更新inode上的访问时间。
    noatime  禁止更新访问时间以加速文件操作，并可以阻止硬盘不断被从休眠状态中唤醒，
             并能达到节省能源和延长硬盘寿命的好处。
    relatime 类似于 noatime 但仅在文件的修改时间晚于访问时间时才更新访问时间。这是默认值。
             它可以让需要知道文件在最后一次被修改后是否被访问过的程序正常工作。

    show_sys_files
    显示"系统"属性的文件，默认不显示。由于Glibc的bug，即使使用此选项，"$MFT"也可能不可见。
        无论show_sys_files是否开启，所有文件都可以直接通过文件名进行访问。
        例如 ls -l '$UpCase' 命令总能够成功执行。

    max_read=value
    设置最大读操作的尺寸，默认为无限。

    silent
        除非使用了 uid, gid, umask, fmask, dmask 选项之一开启了对访问权限的处理，
        否则即使chown/chmod无法正确执行也不报错。这个选项默认开启。

    streams_interface={none|windows|xattr}
    此选项控制用户如何处理NTFS备选数据流(ADS, Alternate Data Streams)特性。
        ADS让一个文件存在不同的数据流，这个特性平常几乎不被使用，但几乎总被恶意程序利用。
        none 表示禁用ADS特性，每个文件的备选数据流都将被忽视。
        windows 表示正常开启ADS特性，用户可以像在Windows环境中一样使用这个特性(例如 cat file:stream)
        xattr 表示将ADS特性映射到 xattr ，用户可以通过 {get,set}fattr 工具操作备选数据流。这是默认值。

    efs_raw  以原始(raw)数据流方式访问NTFS文件系统，这个选项只应该在备份或者恢复的场合下使用。
             它会改变文件的显示尺寸和读写操作的方式，这样加密文件可以在备份或者恢复时不必先进行解密。
             关联到文件的user.ntfs.efsinfo扩展属性也被保存和恢复。

    debug      让ntfs-3g不从终端脱离，而是打印出许多驱动调试信息。
    no_detach  和debug一样，只是输出的信息较少。



reiserfs

       conv   告诉3.6版的reiserfs软件要挂载的是3.5版的文件系统，对于新创建的对象使用3.6版的格式。
              这个文件系统不再与3.5版的工具兼容。

       hash={rupasov|tea|r5|detect}
              选择使用哪种hash函数来在目录内查找文件：

              rupasov
                     不要使用这个选项，因为这种方法可能带来很高概率的hash碰撞。

              tea    随机性高，碰撞较少，但CPU占用率较高。可以在r5散列遇到EHASHCOLLISION错误时使用。

              r5     默认值，也是最好的选择，只要文件系统目录树不是那么大，没有特殊的文件名模式。

              detect 指示mount检测正在使用哪种散列函数，将信息写入reiserfs超级块。
                     这个选项只在第一次挂载旧格式的文件系统时才有用。

       hashed_relocation
       no_unhashed_relocation
              调整块分配器。某些情况下可以带来性能提高。

       noborder
              禁止 border allocator 算法，某些情况下可以带来性能提高。

       nolog  禁止文件系统日志。某些情况下可以带来性能轻微提高，代价是失去了从崩溃中快速恢复的能力。
              即使使用这个选项，reiserfs 仍然进行所有日志动作，将实际的写入保存到日志区域。
              nolog 的实现工作还在开发中。

       notail 默认情况下，reiserfs 将小文件和"文件零头(file tails)"直接保存在树中。
              这样做会给某些工具带来麻烦，例如LILO(8)。这个选项用来禁止将文件放入树中。

       replayonly
              重放日志中的事务，但不真正挂载文件系统。主要由 reiserfsck 使用。

       resize=number
              remount 的一个选项，允许在线扩展reiserfs分区。指示reiserfs假定设备上有number个块。
              这个选项被设计为用于逻辑卷管理(LVM)下的设备。

       user_xattr
              支持"user."扩展属性，参见attr(5)手册页。

       acl    是否支持POSIX Access Control Lists特性，参见acl(5)手册页。

       barrier=none / barrier=flush
              开启/关闭barrier特性。barrier=0 关闭，barrier=1 开启。reiserfs 默认关闭此特性。
              磁盘上配有内部缓存，以便重新调整批量数据的写操作顺序，优化写入性能，
              因此文件系统必须在日志数据写入磁盘之后才能写 commit 记录，
              若 commit 记录写入在先，而日志有可能损坏，那么就会影响数据完整性。
              barrier 强制日志以正确的次序提交到磁盘，这样就可安全的使用磁盘上的内部缓存，代价是降低一些性能。
              除非你的硬盘有后备电池之类的装载，请务必开启 barrier 特性，否则意外掉电可能会造成数据错误。


xfs
       allocsize=size
              设置在执行延时分配写入操作时，文件尾预分配的IO缓冲区尺寸(默认64k)。
              有效值是页大小(一般是4k)的2的指数倍，最大值是"1g"。比如：allocsize=16m

              默认的尺寸是根据一系列文件系统行为进行动态预测的。
              如果使用此选项指定一个明确的值(对于RAID建议设为条带大小的整数倍)，那么将会关闭动态预测算法。

       attr2|noattr2
              启用/禁用启用一个"投机性"改进：在磁盘上存储内联扩展属性。
              默认值取决于磁盘上的特征比特位。建议启用此特性以提升性能。

       barrier|nobarrier
              开启/关闭 barrier 特性。默认为开启(barrier)。
              磁盘或RAID卡上配有内部缓存，以便重新调整批量数据的写操作顺序，优化写入性能，
              因此文件系统必须在日志数据写入磁盘之后才能写 commit 记录，
              若 commit 记录写入在先，而日志有可能损坏，那么就会影响数据完整性。
              barrier 强制日志以正确的次序提交到磁盘，这样就可安全的使用磁盘上的内部缓存，代价是降低一些性能。
              除非你的磁盘或RAID卡有后备电池之类的装置，或者配有UPS，或者像笔记本一样带有后备电池，否则切勿关闭 barrier 特性。

       discard|nodiscard
              是否发出要求块设备回收无用空间的命令。默认值是"nodiscard"。
              此选项对于SSD(固态硬盘)、thin-provision LUN(CONFIG_DM_THIN_PROVISIONING)、虚拟机镜像比较有意义。
              [注意]由于开启此项后对性能有明显的不利影响，所以推荐使用 Util-Linux 包中的 fstrim 程序进行回收。

       grpid|nogrpid
              定义新创建的文件应该获得怎样的GID。默认值是"nogrpid"。
              grpid 表示使用其父目录的GID；
              nogrpid 的含义如下：
                     (1)如果新建文件的父目录没有 setgid 位，那么使用创建此文件的进程的GID。
                     (2)如果新建文件的父目录带有 setgid 位，那么使用其父目录的GID；
                     (3)并且如果新建的是目录时，还要同样设置 setgid 位。

       filestreams
              让数据分配器使用文件流分配模式跨越整个文件系统，
              而不是仅仅局限于使用分配好的目录。

       ikeep|noikeep
              ikeep 表示不删除空白的 inode 集群，而是将它们依然保留在磁盘上。
              noikeep 表示将空白的 inode 集群作为自由空间回收使用。
              默认值是"noikeep"。

       inode32|inode64
              inode32 表示仅创建32位的 inode 。这主要是出于兼容老旧系统的原因。
              inode64 允许在文件系统的任意位置创建 inode，包括那些需要64位 inode 编号的位置。
              默认值是"inode64"。

       largeio|nolargeio
              系统调用stat(2)报告的"st_blksize"值指明了磁盘IO操作的最佳块大小。强制写入小于"st_blksize"大小的数据块时，
              会导致低效的"读取-更改-回写"操作(例如在新式的4k扇区硬盘上，写入1k数据，实际上是"读取4k-更改数据-回写4k")。
              nolargeio 表示系统调用stat(2)尽可能报告较小的"st_blksize"值(通常等于内存页的大小，因为这是页缓存的最小粒度)，
                        以让应用程序避免低效的"读取-更改-回写"操作。"nolargeio"也是默认值。
              largeio 表示这样报告"st_blksize"的值：
                      (1)在指定了"swidth"值的文件系统上，等于"swidth"的值；
                      (2)在未指定"swidth"值、但指定了"allocsize"值的文件系统上，等于"allocsize"的值；
                      (3)如果"swidth"和"allocsize"都未指定，那么其行为和"nolargeio"完全一样。

       logbufs=value
              设置内存中的日志缓存块数量。取值范围是2-8之间。默认值是"8"。
              数字越大性能越好，但是在内存紧张的嵌入式系统上，可以考虑减小此值。

       logbsize=value
              设置内存中每个日志缓存块的大小。取值范围是：16k,32k,64k,128k,256k
              该值必须是格式化时设置的日志条带单元大小(-l su=?k)的整数倍。
              加大日志缓存能提高有大量I/O密集型工作负载的系统性能表现，除非是内存紧张的嵌入式系统，否则应该尽可能设为最大值。
              默认值是 32k 与"日志条带单元大小"(log_sunit)中的较大者。

       logdev=device
       rtdev=device
              使用外部元数据日志和/或实时设备。
              一个XFS文件系统最多可由3部分组成：data, log(可与data分离), real-time(可选)。

       noalign
              分配数据时不按照条带单元的边界进行对齐。若无特别的理由，不应使用此选项。
              此选项仅适用于 mkfs 在格式化时使用了非零的数据对齐参数(sunit, swidth)所创建的文件系统。

       norecovery
              在挂载文件系统时不执行日志恢复(log recovery)操作。
              如果文件系统曾经被异常卸载(如掉电)，那么使用此选项将会导致文件系统出现不一致，
              比如一些文件或目录将无法访问。
              使用此选项的同时必须以只读模式挂载文件系统，否则挂载将会失败。

       nouuid 在挂载文件系统时不检查 uuid 是否重复。
              这在挂载LVM快照时很有用(通常与"norecovery"联用以挂载只读LVM快照)。

       noquota
              强制关闭所有文件系统限额功能

       uquota/usrquota/uqnoenforce/quota
              启用针对单个用户的磁盘限额功能。需要内核开启 CONFIG_XFS_QUOTA 选项。详见 xfs_quota(8) 手册页。

       gquota/grpquota/gqnoenforce
              启用针对用户组的磁盘限额功能。需要内核开启 CONFIG_XFS_QUOTA 选项。详见 xfs_quota(8) 手册页。

       pquota/prjquota/pqnoenforce
              启用针对项目(文件夹)的磁盘限额功能。需要内核开启 CONFIG_XFS_QUOTA 选项。详见 xfs_quota(8) 手册页。

       sunit=value
       swidth=value
              用于指定RAID设备或者条带卷的条带单元(sunit)与条带宽度(swidth)。
              此选项仅适用于 mkfs 在格式化时使用了非零的数据对齐参数(sunit, swidth)所创建的文件系统。
              "value"值的含义是512字节(一个扇区)的倍数，必须是正整数。

              所设定的 sunit/swidth 值必须符合文件系统的实际对齐特征。
              也就是说，合法的 sunit 值必须以2的指数倍增长(最小=1)；而合法的 swidth 值又必须是 sunit 的整数倍。
              [注意]必须同时指定或同时忽略这两个选项，不应该只单独设置其中之一。

              通常仅在文件系统创建之后，底层RAID的磁盘排列又发生改变的情况下，才需要使用此选项。
              例如，向一个 RAID5 LUN 中新增一块硬盘，并重新整理(reshape)之后。

       swalloc
              在当前文件的尾部被扩展，并且文件的大小超过条带宽度的时候，数据的分配将会被向上取整到条带宽度的边界。

       wsync  强制以同步方式执行所有文件系统名字空间的操作。
              这可以确保在文件系统名字空间操作(create, unlink 等)完成之后，所作变更立即被永久写入磁盘中。
              此选项对于高可用群集(HA)环境很有意义。在HA环境中，失败切换(failover)不应被客户端感知，
              也就是说，客户端不应该在失败切换的过程中以及完成之后，看到不一致的名字空间。


iso9660
       ISO9660 是一种标准，描述了用于 CD-ROM 的文件系统结构。
       这种文件系统类型也在一些 DVD 中出现。另外参见 udf 文件系统。

       通常 iso9660 文件名以8.3格式出现(对文件名长度的限制与DOS相同)，另外所有字符都是大写。
       没有文件所有者，权限位，链接数等等，也没有对块设备/字符设备作出扩展。

       RockRidge 是对 iso9660 的扩展，提供了所有这些 unix 文件系统的特性。
       使用 RockRidge 的时候，每个目录记录中都有扩展域来提供这些附加信息。
       这样的文件系统与普通的UNIX文件系统没有什么区别(当然，它是只读的)。

       Joliet 是 Microsoft 对 iso9660 的扩展，允许在文件名中使用 Unicode 字符，也允许长文件名。

       norock
       nojoliet
              禁止RockRidge/Joliet扩展特性，即使它们确实存在。

       check={r[elaxed]|s[trict]}
            relaxed 表示对文件名进行大小写无关的匹配。
            strict 表示对文件名进行严格的大小写敏感匹配，这是默认值。

       uid=value
       gid=value
              设置文件系统上所有文件的UID/GID.可能会覆盖RockRidge扩展中找到的信息。
              默认值皆为'0'

       map={n[ormal]|o[ff]|a[corn]}
              对于非RockRidge的卷：
              normal 将文件名统一转换成小写字母，丢弃结尾的';1'并将';'转换成'.'，这是默认值。
              off 不对文件名做任何转换
              acorn 与 normal 类似，但是如果存在Acorn扩展就应用它。

       mode=value
              对于非RockRidge的卷，指定卷上所有文件/目录的权限(8进制值)。
              默认值是所有人都有读权限。

       unhide 显示隐藏文件(如果常规文件与隐藏文件同名，那么常规文件将不可见，这通常不是你想要的)

       block={512|1024|2048}
              明确指定块的大小，默认值是 block=1024

       cruft  尝试挂载被错误格式化的CD。如果文件长度的高位字节中包含垃圾信息(被错误格式化)，
              使用此选项后，将忽略高位字节，CD依然可以被挂载，但是文件的大小就只能不超过16MB了。

       session=x
              设置多区段 CD 中的区段号。

       sbsector=xxx
              区段从 xxx 扇区开始。

       下列选项仅用于 Microsoft Joliet 扩展：

       iocharset=value
              用于将Unicode字符转换到ASCII字符时使用的字符集(Joliet文件名以Unicode格式存储)。
              反对使用"iocharset=utf8"，推荐使用utf8选项。默认值是 iso8859-1

       utf8   指定Unicode文件名的存储编码为UTF-8格式。


udf
       统一光盘格式(Universal  Disk Format)采用标准的封包写入(Packet Writing)技术，
       将CD-R当作硬盘来使，用户可以在光盘上修改和删除文件。
       其基本原理是在进行烧录时先将数据打包，并在内存中临时建立一个特殊的文件目录表，
       同时接管系统对光盘的访问。被删除的文件或文件中被修改的部分其实仍存在CD-R光盘中，
       修改后的部分则以单独的数据块写入光盘，只不过在内存的目录表中，
       通过设定允许和不允许访问以及特殊链接等重定向寻址方法将数据重新组合，
       让系统找不到“老数据”，或让新数据替换老数据，从而达到删除与修改的目的。
       不过，在增加便利性的同时UDF也减少了有效存储空间，而且还要事先将CD-R与CD-RW盘片进行格式化，
       其中CD-RW盘片格式化后的容量要减少近100MB

       gid=   设置默认的GID
       uid=   设置默认的UID
       umask= 设置默认的umask(8进制)
       mode=  设置默认的文件权限
       dmode= 设置默认的目录权限
       bs=    设置块大小(默认为2048字节)

       unhide   显示隐藏文件
       undelete 显示已经被删除的文件

       adinicb   在inode中嵌入数据(默认)
       noadinicb 不在inode中嵌入数据

       shortad   使用短ad
       longad    使用长ad(默认)

       nostrict  取消严格一致性

       iocharset 设置NLS字符集

       novrs     跳过卷序列识别，用于排错
       session=  设置CDROM session(从"0"开始计数)，默认值是上一次的session值。
       anchor=   设置标准的anchor位置，默认值是256
       lastblock= 设置文件系统最后的block


LOOP设备
       可以通过下面的命令挂载一个LOOP设备：

              mount /tmp/disk.img /mnt -t vfat -o loop=/dev/loop3

       该命令会将 /tmp/disk.img 文件安装为 /dev/loop3 设备，然后再将它挂载到 /mnt

       如果没有明确指定loop设备，而只是给了个'-o loop'选项，例如：

              mount /tmp/disk.img /mnt -o loop

       mount 将会尝试自己寻找一个未被使用的loop设备(可能是 /dev/loop5 这样的)。

       如果连'-o loop'也没有，并且未指定文件系统类型(但能被libblkid识别)，
       那么 mount 将为 /tmp/disk.img 文件自动创建一个loop设备，例如：

              mount /tmp/disk.img /mnt

              mount -t ext3 /tmp/disk.img /mnt

       这种类型的挂载可以识别四个选项: loop, offset, sizelimit, encryption
       事实上，这四个选项都是来自losetup(8)的选项。

       因为 Linux-2.6.25 之后的内核支持loop设备自动销毁(auto-destruction)，
       所以任何由 mount 分配的loop设备都将被 umount 释放，而不必依赖于/etc/mtab文件。

       当然，你也可以手动使用'losetup -d'或'umount -d'来释放loop设备。


返回值
       mount 的返回值可以由下面的值进行比特位叠加

       0      成功

       1      错误的调用或权限

       2      系统错误(内存不够, 不能 fork, 没有更多的loop设备)

       4      mount 内部错误

       8      用户中途打断

       16     写入或锁定 /etc/mtab 失败

       32     挂载失败

       64     部分挂载成功(用于 -a 选项)


帮助程序
       外部挂载帮助程序的语法是：

              /sbin/mount.TYPE spec dir [-sfnv] [-o options] [-t type.subtype]

       TYPE 是文件系统类型，而 -sfnvo 选项的含义与 mount 选项相同。
       -t 选项用于支持文件系统的子类型，例如：/sbin/mount.fuse -t fuse.sshfs

       mount 不会将下列选项传递给 mount.TYPE 帮助程序： unbindable, runbindable, private, rprivate, slave,
       rslave, shared, rshared, auto,  noauto,  comment, x-*, loop, offset, sizelimit
       所有其他选项都以逗号分隔的列表的形式作为 -o 选项的参数传递给 mount.TYPE 帮助程序。


文件
       /etc/fstab        文件系统表

       /etc/mtab         已挂载的文件系统列表

       /etc/mtab~        锁文件

       /etc/mtab.tmp     临时文件

       /etc/filesystems  一系列将要尝试的文件类型

环境变量
       LIBMOUNT_FSTAB=path
              fstab 文件的位置

       LIBMOUNT_MTAB=path
              mtab 文件的位置

       LIBMOUNT_DEBUG=0xffff
              开启调试输出

BUGS
       挂载一个已经损坏的文件系统时可能会导致系统崩溃。

       只有ext2, ext3, vfat文件系统真正支持同步更新( -o sync 和 -o dirsync )

       -o remount 不一定能改变挂载参数。例如，ext2系列的sb参数，fat系列的gid/umask参数

       仅当设备的 label 或 uuid 存在于 /proc/partitions 中的时候，才能通过 label 或 uuid 进行挂载。

       /etc/mtab 和 /proc/mounts 的内容可能会不一致，/etc/mtab 的内容取决于 mount 命令行选项。
       /proc/mounts 的内容取决于内核以及其他设置(远端的NFS服务器)。
       mount 命令可能会报告错误的NFS挂载信息，而 /proc/mounts 文件中的信息可靠性更高。

参见
       mount(2),   umount(2),   fstab(5),  umount(8),  swapon(8),  findmnt(8),
       nfs(5),   xfs(5),   e2label(8),   xfs_admin(8),   mountd(8),   nfsd(8),
       mke2fs(8), tune2fs(8), losetup(8)



util-linux                                      August 2015                                        MOUNT(8)
```