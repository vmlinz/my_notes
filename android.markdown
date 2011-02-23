# Android SDK Core #

Notes on android sdk applications development, mainly about application components and anything interesting to android application development.
# Android Framework #

Core native libs for the android framework, notes on understanding of these libs to get an overview of the whole system.
## Binder ##

Binder, the core ipc mechanism for android platform, which is interesting to analyze. It's important to get it known, so I can get better understanding of the android system design ideas.

I have to understand the binder from kernel driver througth to java binder apis.

Binder工作情景：

* 客户端取得服务器代理对象
* 通过代理对象调用服务器的函数或者访问它的变量，然后等待调用返回
* 代理对象把请求通过binder驱动发送到服务器进程
* 服务器进程处理用户请求并把结果通过binder驱动返回给客户端进程
* 客户端收到服务器进程的结果并继续执行

### Binder Driver ###

Binder驱动是android系统的ipc核心，它负责在打开它的进程之间进行数据交换以及对象映射。服务器进程打开binder驱动后会循环等待客户端请求，客户端打开binder驱动并发送请求后，服务器进程完成请求并同步返回到客户端，完成rpc调用。binder的源码在内核的驱动中。

### Service Manager ###

Service manager是一个特殊的服务器进程，它负责管理注册到binder的服务对象。客户端要调用某个服务功能时，首先用service manager的proxy对象调用service manager来查找和获取服务器对象。sm会返回给客户端它请求的服务器对象，然后客服端就可以通过binder驱动继续调用该服务器对象的远程方法。

sm的源码位置：
<br />
`frameworks\base\cmds\servicemanager\service_manager.c`

sm在系统初始化时启动：
<br />
`service servicemanager /system/bin/servicemanager
    user system
    critical
    onrestart restart zygote
    onrestart restart media
`

sm的代码：

* `binder_open(128*1024);`

sm调用binder的open函数打开binder驱动并将binder设备映射到用户内存上

* `binder_become_context_manager(bs);`

sm调用binder的ioctl命令`BINDER_SET_CONTEXT_MGR`将自己注册为context_manager，这样其他进程都可获取它的代理对象，因为它的服务句柄是固定的`0`

* `binder_loop(bs, svcmgr_handler);`

sm调用binder的ioctl命令进入循环，并监视binder设备的数据。

* `svcmgr_handler();`

sm的回调函数，当binder收到来自客户端的请求时，这个回调函数就会解析请求命令并把结果重新写入到binder设备中。客户端就能通过binder设备得到请求的结果，可能是其他服务对象的handle。

下面的代码片段说明了sm在收到注册新服务时的处理过程：

`
case SVC_MGR_ADD_SERVICE:
    s = bio_get_string16(msg, &len);
    ptr = bio_get_ref(msg);
    if (do_add_service(bs, s, len, ptr, txn->sender_euid))
	return -1;
    break;
`

### Service Providers ###

注册到service manager的系统服务，客户端从sm获取其代理对象，然后完成rpc调用。

### Service Proxies ###

服务代理对象是客户端从sm通过binder驱动获取到的系统服务的代理对象。当客户端有对应的服务器对象时，客户端可以通过binder驱动透明地远程调用服务器进程提供的服务。

#### 服务代理的原理 ####

下面是客户端请求service服务的过程：

* 客户端想sm请求service
* android系统为客户端建立service代理对象
* 客户端通过service代理对象完成rpc远程调用

#### Android进程环境 ####
##### ProcessState #####
android系统中的任何native进程要想使用binder机制，都必须要创建一个ProcessState对象和IPCThreadState对象。binder机制是整个android系统的基础，所以很有必要研究它的工作原理和方式。

高焕堂先生有一篇《android框架底层结构知多少》的文章，里面有对ProcessState等类的一个大致解释。

ProcessState是一个singleton类，一个进程只能有一个类实例。它的作用是维护当前进程中的所有service代理对象（BpBinder）。它首先打开binder设备文件，然后维护进程中已有的service代理对象。

下面的代码显示了ProcessState如何创建一个service代理对象：

`sp<IBinder> ProcessState::getStrongProxyForHandle(int32_t handle)
{
    sp<IBinder> result;

    AutoMutex _l(mLock);

    handle_entry* e = lookupHandleLocked(handle);

    if (e != NULL) {
	// We need to create a new BpBinder if there isn't currently one, OR we
	// are unable to acquire a weak reference on this current one.  See comment
	// in getWeakProxyForHandle() for more info about this.
	IBinder* b = e->binder;
	if (b == NULL || !e->refs->attemptIncWeak(this)) {
	    b = new BpBinder(handle);
	    e->binder = b;
	    if (b) e->refs = b->getWeakRefs();
	    result = b;
	} else {
	    // This little bit of nastyness is to allow us to add a primary
	    // reference to the remote proxy when this team doesn't have one
	    // but another team is sending the handle to us.
	    result.force_set(b);
	    e->refs->decWeak(this);
	}
    }

    return result;
}
`

##### IPCThtreadState #####

Android进程中创建ProcessState类的过程中会打开binder设备文件，并保存其句柄，以供后面的IPCThreadState使用。也就是说实际和binder设备通信的是IPCThreadState类，PorcessState只是打开设备。请查看相关的代码，理解它的主要功能和实现。

IPCThreadState类是ProcessState类的friend类，所以可以访问到它的私有变量和方法。IPCThreadState类中主要通过talkWithDriver和binder设备通信。

它的主要作用：

* 负责当前进程中所有对binder设备的访问
* IPCThreadState也可以理解成对binder设备访问的封装，这样就不用直接使用ioctl来操作binder设备了。

客户端进程和service进程都需要用到IPCThreadState来完成和binder的通信。service进程初始化完成后会轮询binder设备，当客户段请求通过transact方法到来时，service也通过transact完成请求并通过binder设备返回给客户端。


#### BpBinder ####

通过代码我们可以看出首先是通过IPCThreadState读写binder设备增加中相应binder句柄上的Service的引用计数。然后本地保存代理Service的binder句柄mHandle。客户进程对Service的请求都通过调用BpBinder的transact方法来完成。

status_t BpBinder::transact(
    uint32_t code, const Parcel& data, Parcel* reply, uint32_t flags)
{
    // Once a binder has died, it will never come back to life.
    if (mAlive) {
	status_t status = IPCThreadState::self()->transact(
	    mHandle, code, data, reply, flags);
	if (status == DEAD_OBJECT) mAlive = 0;
	return status;
    }

    return DEAD_OBJECT;
}

#### ServiceManager代理 ####

libbinder中有一个全局函数defaultServiceManager可以是进程在C/C++空间获取到ServiceManager的代理。

`sp<IServiceManager> defaultServiceManager()
{
    if (gDefaultServiceManager != NULL) return gDefaultServiceManager;

    {
	AutoMutex _l(gDefaultServiceManagerLock);
	if (gDefaultServiceManager == NULL) {
	    gDefaultServiceManager = interface_cast<IServiceManager>(
		ProcessState::self()->getContextObject(NULL));
	}
    }

    return gDefaultServiceManager;
}
`

调用getContextObject来得到servicemanager的IBinder。

`sp<IBinder> ProcessState::getContextObject(const sp<IBinder>& caller)
{
    if (supportsProcesses()) {
	return getStrongProxyForHandle(0);
    } else {
	return getContextObject(String16("default"), caller);
    }
}
`

然后系统通过interface_cast<IServiceManager>将IBinder转换为IServiceManager的引用。这里需要参考IServiceManager的源代码。

### IBinder接口 ###

Android为了使开发者透明的使用binder机制而不用直接操作binder设备设计了IBinder接口，可以像下面这样理解它。

* 向Android注册的Service也必须是IBinder（继承扩展IBinder接口）对象。后续文章中我们讨论Service的时候我们会介绍到这方面的内容。
* 客户端得到Service代理对象也必须定义成IBinder（继承扩展IBinder接口）对象。这也是为什么BpBinder就是继承自IBinder。
* 客户端发送请求给服务器进程，调用接口的Service代理对象IBinder接口的transact方法。
* Android系统Binder机制将负责把用户的请求，调用Service对象IBinder接口的onTransact方法。具体实现我们将在以后介绍Service的时候讨论。

### Clients ###

客户端一般是android系统上的普通应用程序，它可以通过和binder通信来远程调用服务器进程的功能。

### Binder For Java ###
# Android Middleware #
# Android Linux Kernel #
# Android Open Source Project #
