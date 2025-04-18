// Merchant.java
@Entity
@Data
public class Merchant {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String contactPhone;
    private String email;
    private String address;
    @Enumerated(EnumType.STRING)
    private MerchantStatus status;
    
    @OneToMany(mappedBy = "merchant", cascade = CascadeType.ALL)
    private List<Dish> menu;
    
    @OneToMany(mappedBy = "merchant")
    private List<Order> orders;
}

// Dish.java
@Entity
@Data
public class Dish {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    private BigDecimal price;
    private String category;
    private String imageUrl;
    private boolean isAvailable;
    
    @ManyToOne
    @JoinColumn(name = "merchant_id")
    private Merchant merchant;
}

// Order.java
@Entity
@Data
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private BigDecimal totalAmount;
    @Enumerated(EnumType.STRING)
    private OrderStatus status;
    
    @ManyToOne
    @JoinColumn(name = "merchant_id")
    private Merchant merchant;
    
    @OneToOne(mappedBy = "order", cascade = CascadeType.ALL)
    private Review review;
}
// MerchantController.java
@RestController
@RequestMapping("/api/merchants")
@RequiredArgsConstructor
public class MerchantController {
    private final MerchantService merchantService;

    // 商家入驻申请
    @PostMapping
    public ResponseEntity<Merchant> register(@RequestBody MerchantRegisterDTO dto) {
        return ResponseEntity.ok(merchantService.register(dto));
    }

    // 商家信息更新
    @PutMapping("/{id}")
    public ResponseEntity<Merchant> updateInfo(@PathVariable Long id, @RequestBody MerchantUpdateDTO dto) {
        return ResponseEntity.ok(merchantService.updateInfo(id, dto));
    }

    // 管理员审核
    @PatchMapping("/{id}/status")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> updateStatus(@PathVariable Long id, @RequestParam MerchantStatus status) {
        merchantService.updateStatus(id, status);
        return ResponseEntity.noContent().build();
    }
}

// OrderController.java
@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor
public class OrderController {
    private final OrderService orderService;

    // 商家获取订单列表
    @GetMapping
    public ResponseEntity<Page<Order>> getMerchantOrders(
            @RequestParam Long merchantId,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return ResponseEntity.ok(orderService.getMerchantOrders(merchantId, page, size));
    }

    // 更新订单状态
    @PatchMapping("/{id}/status")
    public ResponseEntity<Void> updateStatus(
            @PathVariable Long id,
            @RequestParam OrderStatus status) {
        orderService.updateStatus(id, status);
        return ResponseEntity.noContent().build();
    }
}
