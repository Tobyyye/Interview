**什么是一致性 hash？自己实现一致性 hash，会用什么数据结构**

在解决分布式系统中负载均衡的问题时候可以使用Hash算法让固定的一部分请求落到同一台服务器上，这样每台服务器固定处理一部分请求（并维护这些请求的信息），起到负载均衡的作用。

但是普通的余数hash（hash(比如用户id)%服务器机器数）算法伸缩性很差，当新增或者下线服务器机器时候，用户id与服务器的映射关系会大量失效。一致性hash则利用hash环对其进行了改进。

比较通俗的理解

https://www.zsythink.net/archives/1182/





# 一致性Hash算法以及java实现

会用什么数据结构**

`SortedMap`接口声明的方法如下表中所示 - 

| 编号 | 方法                                         | 描述                                                         |
| ---- | -------------------------------------------- | ------------------------------------------------------------ |
| 1    | `Comparator comparator()`                    | 返回调用有序映射的比较器。如果自然排序用于调用映射，则返回`null`。 |
| 2    | `Object firstKey()`                          | 返回调用映射中的第一个键。                                   |
| 3    | `SortedMap headMap(Object end)`              | 返回键小于结束的那些映射条目的有序映射。                     |
| 4    | `Object lastKey()`                           | 返回调用映射中的最后一个键。                                 |
| 5    | `SortedMap subMap(Object start, Object end)` | 返回包含键大于或等于`start`且小于`end`的条目的映射。         |
| 6    | `SortedMap tailMap(Object start)`            | 返回包含键大于或等于`start`的条目的映射。                    |

//原文出自【易百教程】，商业转载请联系作者获得授权，非商业转载请保留原文链接：https://www.yiibai.com/java/java_sortedmap_interface.html 

import java.util.LinkedList;
import java.util.List;
import java.util.SortedMap;
import java.util.TreeMap;

public class ConsistencyHashing {

    // 虚拟节点的个数
    private static final int VIRTUAL_NUM = 5;
     
    // 虚拟节点分配，key是hash值，value是虚拟节点服务器名称
    private static SortedMap<Integer, String> shards = new TreeMap<Integer, String>();
     
    // 真实节点列表
    private static List<String> realNodes = new LinkedList<String>();
     
    //模拟初始服务器
    private static String[] servers = { "192.168.1.1", "192.168.1.2", "192.168.1.3", "192.168.1.5", "192.168.1.6" };
     
    static {
        for (String server : servers) {
            realNodes.add(server);
            System.out.println("真实节点[" + server + "] 被添加");
            for (int i = 0; i < VIRTUAL_NUM; i++) {
                String virtualNode = server + "&&VN" + i;
                int hash = getHash(virtualNode);
                shards.put(hash, virtualNode);
                System.out.println("虚拟节点[" + virtualNode + "] hash:" + hash + "，被添加");
            }
        }
    }
     
    /**
     * 获取被分配的节点名
     * 
     * @param node
     * @return
     */
    public static String getServer(String node) {
        int hash = getHash(node);
        Integer key = null;
        SortedMap<Integer, String> subMap = shards.tailMap(hash);
        if (subMap.isEmpty()) {
            key = shards.lastKey();
        } else {
            key = subMap.firstKey();
        }
        String virtualNode = shards.get(key);
        return virtualNode.substring(0, virtualNode.indexOf("&&"));
    }
     
    /**
     * 添加节点
     * 
     * @param node
     */
    public static void addNode(String node) {
        if (!realNodes.contains(node)) {
            realNodes.add(node);
            System.out.println("真实节点[" + node + "] 上线添加");
            for (int i = 0; i < VIRTUAL_NUM; i++) {
                String virtualNode = node + "&&VN" + i;
                int hash = getHash(virtualNode);
                shards.put(hash, virtualNode);
                System.out.println("虚拟节点[" + virtualNode + "] hash:" + hash + "，被添加");
            }
        }
    }
     
    /**
     * 删除节点
     * 
     * @param node
     */
    public static void delNode(String node) {
        if (realNodes.contains(node)) {
            realNodes.remove(node);
            System.out.println("真实节点[" + node + "] 下线移除");
            for (int i = 0; i < VIRTUAL_NUM; i++) {
                String virtualNode = node + "&&VN" + i;
                int hash = getHash(virtualNode);
                shards.remove(hash);
                System.out.println("虚拟节点[" + virtualNode + "] hash:" + hash + "，被移除");
            }
        }
    }
     
    /**
     * FNV1_32_HASH算法
     */
    private static int getHash(String str) {
        final int p = 16777619;
        int hash = (int) 2166136261L;
        for (int i = 0; i < str.length(); i++)
            hash = (hash ^ str.charAt(i)) * p;
        hash += hash << 13;
        hash ^= hash >> 7;
        hash += hash << 3;
        hash ^= hash >> 17;
        hash += hash << 5;
        // 如果算出来的值为负数则取其绝对值
        if (hash < 0)
            hash = Math.abs(hash);
        return hash;
    }
     
    public static void main(String[] args) {
        
        //模拟客户端的请求
        String[] nodes = { "127.0.0.1", "10.9.3.253", "192.168.10.1" };
        
        for (String node : nodes) {
            System.out.println("[" + node + "]的hash值为" + getHash(node) + ", 被路由到结点[" + getServer(node) + "]");
        }
        
        // 添加一个节点(模拟服务器上线)
        addNode("192.168.1.7");
        // 删除一个节点（模拟服务器下线）
        delNode("192.168.1.2");
     
        for (String node : nodes) {
            System.out.println("[" + node + "]的hash值为" + getHash(node) + ", 被路由到结点[" + getServer(node) + "]");
        }
    }
}
————————————————
版权声明：本文为CSDN博主「白开水Luis」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/zhanglu0223/article/details/100579254