# test
}
}
}

}
}

}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}

}
}
}
}
}
}
}
}
}
}
}
}
}

}

}
}

}
}

}

}

}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}
}

}

}
}
}
}
}
}
}
}
}

 /**
  * 读service.txt配置文件
  * 
  * @return
  * @throws Exception
  */
 public static Profile init() throws Exception {
  // start service
  String root = Platform.getRoot();
  // config
  Profile service = new Profile(root + "service.txt");

  return service;
 }

 /**
  * stop the startup
  * 
  * @throws Exception
  */
 public static void stop() throws Exception {
  try {
   Profile service = init();
   // 通过IP识别环境信息，必须在初始化日志之前识别环境
   String envIPList = service.get("system", "EnvIPList");
   System.setProperty("Bserver.EnvIPList", envIPList);

   final ILogger log = LogUtils.getLogger(Bserver.class);

   log.info("Bserver begin to shutting down.");

   String shutdownPortStr = service.get("system", "ShutdownPort");
   if (shutdownPortStr == null) {
    log.warn("shutdown port not config in [service.txt] yet, please shutdown Bserver by force.");
    return;
   }
   int shutdownPort = Integer.parseInt(shutdownPortStr);
   Socket client = new Socket("127.0.0.1", shutdownPort);
   Writer writer = new OutputStreamWriter(client.getOutputStream());
   writer.write(SHUTDOWN);
   writer.flush();
   writer.close();
   client.close();
   log.info("Bserver is shutting down, please wait...");
  } catch (Throwable th) {
   throw th;
  } finally {
   LogUtils.shutdownLogger();
  }
 }

 /**
  * flush logger
  * @throws Exception
  */
 public static void flush() throws Exception {
  try {
   Profile service = init();
   // 通过IP识别环境信息，必须在初始化日志之前识别环境
   String envIPList = service.get("system", "EnvIPList");
   System.setProperty("Bserver.EnvIPList", envIPList);

   String shutdownPortStr = service.get("system", "ShutdownPort");
   if (shutdownPortStr == null) {
    System.out.println("shutdown port not config in [service.txt] yet, manual flush logger can not work.");
    return;
   }
   int shutdownPort = Integer.parseInt(shutdownPortStr);
   Socket client = new Socket("127.0.0.1", shutdownPort);
   Writer writer = new OutputStreamWriter(client.getOutputStream());
   writer.write(FLUSHLOG);
   writer.flush();
   writer.close();
   client.close();
   System.out.println("Bserver flushing logger finished, please check the log file.");
  } catch (Throwable th) {
   throw th;
  }
 }

 /**
  * start the startup
  * 
  * @throws Exception
  */
 public static void start() throws Exception {
  String tips = "Bserver is initializing, please wait...";
  // start service
  Profile service = init();

  // 通过IP识别环境信息，必须在初始化日志之前识别环境
  String envIPList = service.get("system", "EnvIPList");
  System.setProperty("Bserver.EnvIPList", envIPList);
  int msgMaxLengthInt = DEFAULT_MAX_LEN;
  String msgMaxLength = service.get("system", "MsgMaxLength");
  if (msgMaxLength != null) {
   msgMaxLengthInt = Integer.parseInt(msgMaxLength) * 1024 * 1024;
  }
  System.setProperty("Bserver.MsgMaxLength", String.valueOf(msgMaxLengthInt));

  ILogger log = LogUtils.getLogger(Bserver.class);
  log.info(tips);
  tips = "*************************设置系统参数[read service.txt]****************";
  log.info(tips);
  log.info("Bserver env IP list is " + envIPList);

  // 设置dbconfig.xml里面password解码密钥
  String secretKey = service.get("system", "SecretKey");
  if (secretKey != null) {
   System.setProperty("Bserver.SecretKey", secretKey);
   log.info("Bserver secret key is " + secretKey);
  }

  int sockPort = Integer.parseInt(service.get("system", "Port"));

  log.info("Bserver socket port is " + sockPort);

  // 设置系统代号，托管核心或者托管银行
  Platform.setBsrNo(service.get("system", "BsrNo"));

  log.info("Bserver NO. is " + Platform.getBsrNo());

  log.info("Bserver Runmode is " + Platform.getRunMode().toString());
        
  AppContext.getApplicationContext().setServerFile(service);//将service.txt文件放入内存中，以便各系统获取

  tips = "*************************设置[Worker]线程池参数 ************************";
  log.info(tips);
  int workerThreadNum = 0;
  String strWorkerThreadNum = service.get("system", "WorkerThreadNum");
  if (strWorkerThreadNum == null) {
   tips = "WorkerThread option[WorkerThreadNum] is not config in service.txt, " 
        + "use default value 2*CPU";
  } else {
   workerThreadNum = Integer.parseInt(strWorkerThreadNum);
   tips = "WorkerThread option[WorkerThreadNum] is " + strWorkerThreadNum;
  }
  log.info(tips);

  tips = "*************************设置[BUSINS]业务线程池参数 *********************";
  log.info(tips);
  // BUSINS线程池参数[maxThreadNum最大线程数]
  int maxThreadNum = 100;
  // BUSINS线程池参数[coreThreadNum核心线程数]
  int coreThreadNum = 30;
  // BUSINS线程池参数[queueCapacity工作队列大小]
  int queueCapacity = 50;
  String strMaxThreadNum = service.get("system", "MaxThreadNum");
  if (strMaxThreadNum == null) {
   tips = "BusinsThread option[MaxThreadNum] is not config in service.txt, " 
        + "use default value " + maxThreadNum;
  } else {
   maxThreadNum = Integer.parseInt(strMaxThreadNum);
   tips = "BusinsThread option[MaxThreadNum] is " + strMaxThreadNum;
  }
  log.info(tips);
  String strCoreThreadNum = service.get("system", "CoreThreadNum");
  if (strCoreThreadNum == null) {
   tips = "BusinsThread option[CoreThreadNum] is not config in service.txt, " 
        + "use default value " + coreThreadNum;
  } else {
   coreThreadNum = Integer.parseInt(strCoreThreadNum);
   tips = "BusinsThread option[CoreThreadNum] is " + strCoreThreadNum;
  }
  log.info(tips);
  String strQueueCapacity = service.get("system", "QueueCapacity");
  if (strQueueCapacity == null) {
   tips = "BusinsThread option[QueueCapacity] is not config in service.txt, " 
         +"use default value " + queueCapacity;
  } else {
   queueCapacity = Integer.parseInt(strQueueCapacity);
   tips = "BusinsThread option[QueueCapacity] is " + strQueueCapacity;
  }
  log.info(tips);
  tips = "*************************初始化数据源和数据库连接池[DataSources]***********";
  String connectDB = service.get("system", "ConnectDB");
  boolean isConnectDB = (connectDB == null) ? true : connectDB.equals("true");
  if (isConnectDB) {
   log.info(tips);
   DataSources.initialize();
   log.info("Bserver Database URL:" + System.getProperty("Bserver.database.url"));
   log.info("Bserver create[DataSources] success.");
  }

  tips = "*************************初始化业务调度类[Dispatcher]*********************";
  log.info(tips);
  Platform.createDispatcher(service);
  log.info("Bserver create[Dispatcher] success," 
  + Platform.getDispatcher().getName());

  tips = "*************************注册事件监听器[Listener]*************************";
  if (service.containSection("Listener")) {
   log.info(tips);
   String appContextEventListeners = service.get("Listener", "AppContextEventListener");
   String businsContextEventListeners = service.get("Listener", "BusinsContextEventListener");
   AppContext.setAppContextListener(appContextEventListeners);
   AppContext.setBusinsContextListener(businsContextEventListeners);
   log.info("Bserver register[Listener] success.");
  }
  AbstractAppInitializer initer;
  String hi = service.get("quartz", "Initializer");
        if (hi != null)
        {
            initer = (AbstractAppInitializer) Class.forName(hi).newInstance();
        }
        else
        {
            initer = new DefaultAppInitializer();
        }
        initer.init(service);
        
  /*********************************************************************************
   * Configure the startup. <br>
   * BOSSER线程池 - 接收TCP连接请求,处理socket连,accept socket connection
   * <p>
   * WORKER线程池 - 处理IO事件,例如TCP/IP数据流的报文解码和编码
   * <p>
   * BUSINS线程池 - 业务处理线程，与数据数据库交互，执行业务处理逻辑。
   * <p>
   * 每个线程池只关注自己的任务，当前任务完成即释放线程资源。
   *********************************************************************************/

  log.info("*********************开始启动[BOSSER|WORKER|BUSINS]三个工作线程池**********");
  // BOSS线程池接，起1个单线程
  EventLoopGroup bossGroup = new NioEventLoopGroup(1,
    new DefaultThreadFactory("BOSSER", false, Thread.MAX_PRIORITY));
  // WORKER线程池,线程数默认值是CPU的个数2倍
  EventLoopGroup workerGroup = new NioEventLoopGroup(workerThreadNum,
    new DefaultThreadFactory("WORKER", false, Thread.MAX_PRIORITY));
  // 业务处理线程，核心线程数通过参数配置
  int corePoolSize = coreThreadNum;
  // 业务处理线程，最大线程数通过参数配置
  int maximumPoolSize = maxThreadNum;
  // 最大闲置时间10分钟，超过core数量的闲置线程会销毁
  long keepAliveTime = 10 * 60;
  int blockQueueCapacity = queueCapacity;
  if (corePoolSize > maximumPoolSize) {
   throw new Exception("business thread pool config error,CoreThreadNum can't more than MaxThreadNum,"
     + "please check option[CoreThreadNum,MaxThreadNum] in file service.txt.");
  }
  /**
   * BUSINS线程池工作原理
   * <p>
   * 1.如果正在运行的线程数量小于 coreThreadNum，那么马上创建线程运行这个任务；
   * <p>
   * 2.如果正在运行的线程数量大于或等于 coreThreadNum，那么将这个任务放入队列；
   * <p>
   * 3.如果这时候队列满了，而且正在运行的线程数量小于 maxThreadNum，则创建新线程运行这个任务；
   * <p>
   * 4.如果队列满了，而且正在运行的线程数量大于或等于maxThreadNum，那么线程池会抛出系统繁忙的异常；
   **/
  ExecutorService executor = new ThreadPoolExecutor(
  corePoolSize, 
  maximumPoolSize, 
  keepAliveTime,
  TimeUnit.SECONDS, 
  new LinkedBlockingQueue<Runnable>(blockQueueCapacity),
  new DefaultThreadFactory("BUSINS", false, Thread.NORM_PRIORITY), 
  new AbortPolicy());

  try {
   ServerBootstrap b = new ServerBootstrap();
   b.group(bossGroup, workerGroup)
   .channel(NioServerSocketChannel.class)
   .handler(new LoggingHandler())
   /**** 以下设置设置TCP参数 ****/
   // BACKLOG标识当服务器请求处理线程全满时，用于临时存放已完成三次握手的请求的队列的最大长度。
   .option(ChannelOption.SO_BACKLOG, 1000)
   // SO_TIMEOUT表示输入流读取数据read方法的最长等待时间。一旦超过设置的SO_TIMEOUT，将抛出超时异常。
   .option(ChannelOption.SO_TIMEOUT, TIME_OUT)
   // SO_KEEPALIVE是否启用TCP心跳机制。
   .option(ChannelOption.SO_KEEPALIVE, false)
   .childHandler(new ChannelRegister(executor, msgMaxLengthInt));

   // Start the startup.
   ChannelFuture f = b.bind(sockPort).sync();

   Channel ch = f.channel();

   log.info("ThreadPool[BOSSER|WORKER|BUSINS] starting success.");

   tips = "***********************设置关闭应用监听端口[shutdown listener]******************";
   String shutdownPortStr = service.get("system", "ShutdownPort");
   if (shutdownPortStr == null) {
    log.info("option[ShutdownPort] not exists in service.txt, "
      + "shutdown Bserver gracefully is unavailable.");
   }
   // 设置关闭监控线程
   else {
    log.info(tips);
    int shutdownPort = Integer.parseInt(shutdownPortStr);
    new Stop$FlushServer(ch, shutdownPort).start();
    log.info("Bserver shutdown listener starting success,shutdown port is "
    + shutdownPortStr);
   }

   log.info("Bserver starting success.");

   // 非开发环境，启动完毕后关闭控制台日志
   Platform.setConsoleOuput();
   // 当日志文件缓存开启，启动信息手动刷到日志文件
   LogUtils.flushRootLogger();

   // Wait until the startup socket is closed.
   ch.closeFuture().sync();
   
  } finally {
   // Shut down all event loops to terminate all threads.
      log.info("Bserver is shutting down.");
   executor.shutdownNow();
   bossGroup.shutdownGracefully();
   workerGroup.shutdownGracefully();
   LogUtils.shutdownLogger();
  }

 }

 /**
  * 创建ServerSocket监听端口，关闭main线程的 ServerSocketChannel,唤醒main线程做其他线程的退出。
  * 
  * based on tomcat8 shutdown,you may found more detains in APACHE tomcat8
  */
 public static class Stop$FlushServer extends Thread {

  private Random random;

  private int shutdownPort;

  Channel ch;

  public Stop$FlushServer(Channel ch, int shutdownPort) {
   this.ch = ch;
   this.setDaemon(true);
   this.setName("SHUTDW");
   this.shutdownPort = shutdownPort;
  }

  public void run() {
   // Wait for the next connection
   ILogger log = LogUtils.getLogger(Bserver.class);
   ServerSocket serverSocket = null;
   try {
    serverSocket = new ServerSocket(shutdownPort, 1, InetAddress.getByName("localhost"));
   } catch (Exception e1) {
    log.warn(e1);
   }
   Socket socket = null;
   while (serverSocket != null) {
    StringBuilder command = new StringBuilder();
    try {
     InputStream stream;
     long acceptStartTime = System.currentTimeMillis();
     try {
      socket = serverSocket.accept();
      socket.setSoTimeout(10 * 1000); // Ten seconds
      stream = socket.getInputStream();
     } catch (SocketTimeoutException ste) {
      // This should never happen but bug 56684 suggests that
      // it does.
      log.warn("Bserver.accept.timeout", Long.valueOf(System.currentTimeMillis() - acceptStartTime),
        ste);
      continue;
     } catch (AccessControlException ace) {
      log.warn("Bserver.accept security exception: " + ace.getMessage(), ace);
      continue;
     } catch (IOException e) {
      log.error("Bserver.await: accept: ", e);
      break;
     }
     // Read a set of characters from the socket
     int expected = 1024; // Cut off to avoid DoS attack
     while (expected < SHUTDOWN.length()) {
      if (random == null)
       random = new Random();
      expected += (random.nextInt() % 1024);
     }
     while (expected > 0) {
      int ch = -1;
      try {
       ch = stream.read();
      } catch (IOException e) {
       log.warn("Bserver.await: read: ", e);
       ch = -1;
      }
      if (ch < 32) // Control character or EOF terminates loop
       break;
      command.append((char) ch);
      expected--;
     }
    } finally {
     // Close the socket now that we are done with it
     try {
      if (socket != null) {
       socket.close();
      }
     } catch (IOException e) {
      // Ignore
     }
    }

    // Match against our command string
    String commandStr = command.toString();
    if (commandStr.equals(SHUTDOWN)) {
     log.info("Bserver.shutdown via shutdown Port");
     Channel ch = this.ch;
     ch.unsafe().close(ch.unsafe().voidPromise());
     break;

    } else if (commandStr.equals(FLUSHLOG)) {
     log.info("Bserver manual flush logger cache via shutdown Port");
     LogUtils.flushAllLogger();

    } else
     log.warn("Bserver.await: Invalid command '" + command.toString() + "' received");
   }
  }
 }

}
