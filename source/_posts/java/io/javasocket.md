---
title: socket
date: 2018-09-26 17:49:23
tags: [java]
---

#### 1.java.net.socket
server端代码，使用子线程处理连接.
###### (1)存储消息的队列
使用两个队列InMsgQueue和OutMsgueue来存储输入输出数据。
```
/**
 *客户端返回数据队列
 */
public class InMsgQueue {
    /**
     * 阻塞队列，存储返回的数据
     */
    private Map<String, LinkedBlockingQueue<String>> ins ;


    private static final InMsgQueue instance = new InMsgQueue();

    private InMsgQueue(){
        ins =  new HashMap<String, LinkedBlockingQueue<String>>();
    }

    public static final InMsgQueue getInstance(){
        return instance;
    }

    public Map<String, LinkedBlockingQueue<String>> getDatas(){
        return this.ins;
    }

}

```
OutMsgQueue:
```
/**
 * 维护要执行的命令
 */
public class OutMsgQueue {

    /**
     * 阻塞队列，存储页面输入的命令
     */
    private Map<String, LinkedBlockingQueue<String>> outs ;


    private static final OutMsgQueue instance = new OutMsgQueue();

    private OutMsgQueue(){
        outs =  new HashMap<String, LinkedBlockingQueue<String>>();
    }

    public static final OutMsgQueue getInstance(){
        return instance;
    }

    public Map<String, LinkedBlockingQueue<String>> getDatas(){
        return this.outs;
    }

}

```



###### (2)sockerServer
```
/**
 * 单例创建全局socketserver
 */
public class SocketServer {

    private ServerSocket server = null;
    private int port = 12354;
    private final static int minPort = 1023;
    private final static int maxPort = 65535;
    private static final  SocketServer instance = new SocketServer();

    private SocketServer(){
        try {
            if(port >= minPort && port <= maxPort){
                server = new ServerSocket(port);
            }else {
                server = new ServerSocket(12354);
            }
        }catch (IOException e){
            System.out.println("sockerserver 创建失败 " + e.getMessage());
        }
    }

    public static final SocketServer getInstance(){
        return instance;
    }

    public ServerSocket getServer(){
        return this.server;
    }

    public void setPort(int port){
        this.port = port;
    }
    public int getPort(){
        return this.port;
    }
```

###### (3)处理链接
```

/**
 * 处理单个socket的服务类
 * 普通的socket消息收发,通过阻塞队列与页面进行交互
 */
public class SocketThread implements Runnable{


    private Socket socket = null;
    /**
     * 每个socket的对应id
     */
    private String id;

    public SocketThread(Socket socket){
        this.socket = socket;
        this.id = socket.getInetAddress().getHostAddress() + ":" + socket.getPort();
    }
    @Override
    public void run(){
        Map<String, LinkedBlockingQueue<String>> mapIn = InMsgQueue.getInstance().getDatas();
        Map<String, LinkedBlockingQueue<String>> mapOut = OutMsgQueue.getInstance().getDatas();
        LinkedBlockingQueue<String> queueOut = mapOut.get(id);
        LinkedBlockingQueue<String> queueIn = mapIn.get(id);
        try {

            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            OutputStream out = socket.getOutputStream();
            String line = null;
            while (true){
                StringBuilder sb = new StringBuilder();
                // 999999999 结束标志
                while (!(line = in.readLine()).equals("999999999")){
                    sb.append(line);
                }
                System.out.println("[+]: " + sb.toString());
                if(sb.toString().equals("exit()")){
                    break;
                }
                try {
                    queueIn.put(sb.toString());
                }catch (InterruptedException e2){
                    e2.printStackTrace();
                    queueIn.offer(e2.getMessage());
                }
                //读取队列中的命令，send
                try {
                    //阻塞等待页面输入
                    out.write(queueOut.take().getBytes());
                }catch (InterruptedException e1){
                    e1.printStackTrace();
                    out.write("whoami".getBytes());
                }

            }
            in.close();
            out.close();
        }catch (IOException e){
            System.out.println(e.getMessage());
        }finally {
            //删除队列中的连接信息
            mapIn.remove(id);
            mapOut.remove(id);
            try {
                if (socket != null){
                    socket.close();
                    System.out.println("[*]: 异常，关闭socket" + id);
                }
            }catch (IOException e){
                System.out.println("[*]: " + id + " 未正常关闭！ " + e.getMessage());
            }

        }
    }

    public void setSocket(Socket socket){
        this.socket = socket;
    }
```

###### (4)使用多线程处理不同socket
```
**
 * 自动启动socket监听
 */
//控制bean的加载顺序  确保不为null
@DependsOn({"socketLogService","springUtil"})
@Component
public class StartSocket implements ApplicationRunner {

    SocketLogService socketLogService = (SocketLogService)SpringUtil.getBean("socketLogService");

    @Override
    public void run(ApplicationArguments args){
        ServerSocket server = SocketServer.getInstance().getServer();
        ThreadPoolExecutor executor = new ThreadPoolExecutor(4,4,24*60,
                TimeUnit.MINUTES,new LinkedBlockingQueue<>());
        try {
            while (true) {
                Socket socket = server.accept();
                //连接日志
                SocketLogEntity entity = new SocketLogEntity();
                entity.setClientIp(socket.getInetAddress().getHostAddress());
                entity.setClientPort(socket.getPort());
                socketLogService.insertLog(entity);
                //新来一个连接时，更新队列
                Map<String, LinkedBlockingQueue<String>> mapIn = InMsgQueue.getInstance().getDatas();
                Map<String, LinkedBlockingQueue<String>> mapOut = OutMsgQueue.getInstance().getDatas();
                String ip = socket.getInetAddress().getHostAddress() + ":" + socket.getPort();
                if (!mapIn.containsKey(ip)) {
                    mapOut.put(ip, new LinkedBlockingQueue<String>());
                    mapIn.put(ip, new LinkedBlockingQueue<String>());
                }
                //开单独的线程进行处理连接
                executor.execute(new SocketThread(socket));
            }
        }catch (IOException e){
            e.printStackTrace();
        }
    }

}
```

#### 2.nio

#### 3.netty
