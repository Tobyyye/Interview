ReentrantLock

首先ReentrantLock和NonReentrantLock都继承父类AQS，其父类AQS中维护了一个同步状态status来计数重入次数，status初始值为0。



https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html