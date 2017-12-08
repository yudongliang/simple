# simple

/*yudongliang*/

import sun.misc.Queue;

import java.util.concurrent.*;
import java.util.function.Consumer;

/**
 * 消费者工厂
 */
    public class TaskCenterFactory<T,L extends  TaskCenterService>{

        private TaskCenterFactory factory ;

        private ExecutorService threadPool;

        private static final int initTaskCount = 1;

        private static final int maxTaskCount = 50;

        private static final int topicSize = 20000;

        private LinkedBlockingQueue<T> factoryQueue;

        private CountDownLatch countDownLatch = null; //原子操作

        private L l;

        private TaskCenterFactory(L l) {
            if (factoryQueue == null){
                synchronized (this){
                    this.l =l;
                    if (factoryQueue == null){
                        try{
                            factoryQueue = new LinkedBlockingQueue<T>(topicSize );
                            // Tode 读取排位置文件
                            threadPool = Executors.newFixedThreadPool(maxTaskCount);

                            initTask(threadPool);

                        } catch (Exception e){
                            e.printStackTrace();
                        }
                    }
                }

            }

        }

        private void initTask(ExecutorService threadPool){
            for(int i = 0 ; i < maxTaskCount ; i++){
                threadPool.execute(new Consumer(i+""));
            }
        }

        public void factoryBuild(L l) throws Exception{
            if (l == null){

                throw  new Exception("this is null");

            } else if (factory == null){
                synchronized (this){
                    if (factory == null){
                        factory  =  new TaskCenterFactory(l);
                    }else {
                        throw  new Exception("factory is build");
                    }
                }
            } else {
                throw  new Exception("factory is build");
            }
        }
        //入列
        public void pushQueue(T t) throws  Exception{
            factoryQueue.put(t);
        }

        // 阻塞
        private T takeQueue() throws  Exception{
            T t = factoryQueue.take();
            return t;
        }


        class Consumer implements  Runnable{
            private String name;

            private Consumer(String name){
                this.name = name;
            }

            @Override
            public void run() {

                while (true){
                    System.err.print(this.name + "running>>>>>>>>>" + System.currentTimeMillis());
                    try{

                        T t = takeQueue();

                        l.taskEcecute(t); // 自己实现的业务

                    } catch (Exception e){
                        e.printStackTrace();
                    }

                }
            }
        }


        class Properties{
            // 配置文件存放处
        }



    }
    
    // 只开放一个Interface
    public interface TaskCenterService<T>{

        public Object push(T t);

        public void taskEcecute(T t);

        public void callBack(); // 异步回调

    }












