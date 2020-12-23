1. 编写c代码

```c
test.c 代码
    
int testadd(int a, int b ){
    return a + b; 
}

```



2. 编译指令： 

``` bash
gcc -fPIC -shared *.c -std=c99 -o libc_utils.so 
```



3. 查看openresty依赖库位置

``` bash
ldd /usr/local/openresty/nginx/sbin/nginx
        linux-vdso.so.1 =>  (0x00007fec0ab8e000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007fec0a76b000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007fec0a54f000)
        libcrypt.so.1 => /lib64/libcrypt.so.1 (0x00007fec0a318000)
        libluajit-5.1.so.2 => /usr/local/openresty/luajit/lib/libluajit-5.1.so.2 (0x00007fec0a097000)
        libm.so.6 => /lib64/libm.so.6 (0x00007fec09d95000)
        libpcre.so.1 => /usr/local/openresty/pcre/lib/libpcre.so.1 (0x00007fec09b24000)
        libssl.so.1.1 => /usr/local/openresty/openssl111/lib/libssl.so.1.1 (0x00007fec09891000)
        libcrypto.so.1.1 => /usr/local/openresty/openssl111/lib/libcrypto.so.1.1 (0x00007fec093ab000)
        libz.so.1 => /usr/local/openresty/zlib/lib/libz.so.1 (0x00007fec09190000)
        libc.so.6 => /lib64/libc.so.6 (0x00007fec08dc2000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fec0a96f000)
        libfreebl3.so => /lib64/libfreebl3.so (0x00007fec08bbf000)
        libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007fec089a9000)
```



4. lua程序引用

```lua
local ffi = require("ffi")
ffi.cdef[[
int testadd(int a,int b);
]]
local cutils = ffi.load('libc_utils')

ngx.say(cutils.testadd(200,50))
```



参考： 

https://www.javatt.com/p/41770

https://blog.csdn.net/u013139008/article/details/83269548