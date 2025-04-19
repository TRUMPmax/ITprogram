import java.util.*;

public class OrderManager {

    // 模拟订单ID递增
    private static int orderCounter = 1000;

    // 订单状态枚举
    enum OrderStatus {
        CREATED,
        PAID,
        DELIVERING,
        COMPLETED,
        CANCELLED
    }

    // 订单实体类
    static class Order {
        String orderId;
        String userName;
        String merchantName;
        List<String> itemList;
        double totalPrice;
        OrderStatus status;
        Date createTime;

        public Order(String userName, String merchantName, List<String> itemList, double totalPrice) {
            this.orderId = "ORD" + (++orderCounter);
            this.userName = userName;
            this.merchantName = merchantName;
            this.itemList = itemList;
            this.totalPrice = totalPrice;
            this.status = OrderStatus.CREATED;
            this.createTime = new Date();
        }

        public void displayInfo() {
            System.out.println("订单编号：" + orderId);
            System.out.println("下单用户：" + userName);
            System.out.println("商家名称：" + merchantName);
            System.out.println("商品清单：" + itemList);
            System.out.println("总价格：" + totalPrice);
            System.out.println("订单状态：" + status);
            System.out.println("下单时间：" + createTime);
            System.out.println("--------------------------------");
        }
    }

    // 所有订单列表
    private List<Order> orderList = new ArrayList<>();

    // 创建订单方法
    public Order createOrder(String userName, String merchantName, List<String> itemList, double totalPrice) {
        Order newOrder = new Order(userName, merchantName, itemList, totalPrice);
        orderList.add(newOrder);
        System.out.println("订单创建成功！");
        newOrder.displayInfo();
        return newOrder;
    }

    // 根据订单号查询订单
    public Order findOrderById(String orderId) {
        for (Order order : orderList) {
            if (order.orderId.equals(orderId)) {
                return order;
            }
        }
        return null;
    }

    // 更新订单状态
    public boolean updateOrderStatus(String orderId, OrderStatus newStatus) {
        Order order = findOrderById(orderId);
        if (order != null) {
            order.status = newStatus;
            System.out.println("订单状态已更新为：" + newStatus);
            return true;
        } else {
            System.out.println("未找到该订单！");
            return false;
        }
    }

    // 显示所有订单
    public void listAllOrders() {
        if (orderList.isEmpty()) {
            System.out.println("暂无订单信息。");
            return;
        }
        for (Order order : orderList) {
            order.displayInfo();
        }
    }

    // 模拟测试
    public static void main(String[] args) {
        OrderManager manager = new OrderManager();

        // 创建订单
        List<String> items = Arrays.asList("炸鸡", "可乐", "薯条");
        manager.createOrder("小明", "鸡你太美炸鸡店", items, 45.5);

        // 创建第二个订单
        manager.createOrder("小红", "米粉铺子", Arrays.asList("螺蛳粉", "冰绿茶"), 32.0);

        // 查询并更新订单
        manager.updateOrderStatus("ORD1001", OrderStatus.PAID);
        manager.updateOrderStatus("ORD1001", OrderStatus.DELIVERING);

        // 显示所有订单
        manager.listAllOrders();
    }
}
