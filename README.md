# Raspberry-Pi-run-your-kernel-module-at-boot-up
This show steps how to run your own kernel module at bootup.

Just to share what I've learn:
  
  Assume you have a hello.ko kernel module and want the kernel to load it at boot up.
  
  Here is the hello.c, you expect to see "Loading hello module..." in the dmesg after kernel bootup.
  
    #include <linux/module.h>     /* Needed by all modules */
    #include <linux/kernel.h>     /* Needed for KERN_INFO */
    #include <linux/init.h>       /* Needed for the macros */

    static int __init hello_init(void)
    {
        printk(KERN_INFO "Loading hello module...\n");
        return 0;
    }

    static void __exit hello_exit(void)
    {
        printk(KERN_INFO "Goodbye Kernel\n");
    }

    module_init(hello_init);
    module_exit(hello_exit);

    MODULE_LICENSE("GPL");
    MODULE_AUTHOR("test");
    MODULE_DESCRIPTION("Hello Kernel");
    MODULE_VERSION("0.0");

You need to add your hello.ko in /etc/modules for kernel to load it. 

    pi@raspberrypi:~ $ cat /etc/modules
    # /etc/modules: kernel modules to load at boot time.
    #
    # This file contains the names of kernel modules that should be loaded
    # at boot time, one per line. Lines beginning with "#" are ignored.

    i2c-dev
 
 As show above the /etc/modules does not have the hello (your kernel module, don't need to provide .ko).
 Follow the steps below to add your hello to the /etc/modules
 
    sudo cp hello.ko /lib/modules/$(uname -r)			
    echo 'hello' | sudo tee --a /etc/modules > /dev/null			
    sudo depmod -a			
    sudo modprobe hello			
 
 
 The "sudo cp hello.ko /lib/modules/$(uname -r)" will copy the hello.ko to your /lib/modules/{your kernel}
 In my case, my kernel is 4.19.93-v7l+; therefore, the hello.ko will copy to my /lib/modules/4.19.93-v7l+
    
    The uname -r return your raspberrpi kernel that being used.
    pi@raspberrypi:~ $ uname -r
       4.19.93-v7l+

 The "echo 'hello' | sudo tee --a /etc/modules > /dev/null" will write 'hello' to the /etc/modules file.

    pi@raspberrypi:~ $ cat /etc/modules
    # /etc/modules: kernel modules to load at boot time.
    #
    # This file contains the names of kernel modules that should be loaded
    # at boot time, one per line. Lines beginning with "#" are ignored.

    i2c-dev
    hello
    
 After run "sudo depmod -a" and "sudo modprobe hello", it will show the "Loading hello module..." in dmesg.
 
    pi@raspberrypi:~ $ sudo depmod -a
    pi@raspberrypi:~ $ sudo modprobe hello
    pi@raspberrypi:~ $ dmesg | tail -5
    [   13.016337] Bluetooth: RFCOMM TTY layer initialized
    [   13.016351] Bluetooth: RFCOMM socket layer initialized
    [   13.016370] Bluetooth: RFCOMM ver 1.11
    [ 1273.399223] hello: loading out-of-tree module taints kernel.
    [ 1273.399642] Loading hello module...
    pi@raspberrypi:~ $ 
   
  Now reboot your kernel by run "sudo reboot".
  Log on your raspberry pi and run dmesg, as you can see below that the hello.ko is loaded right after the i2c
  
    [    1.901589] i2c /dev entries driver
    [    1.903329] hello: loading out-of-tree module taints kernel.
    [    1.903743] Loading hello module...
    [    2.440012] EXT4-fs (mmcblk0p2): re-mounted. Opts: (null)
    
 Hope this will help

    
    

   
