// 商家实体
@Entity
@Data
public class Merchant {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String contact;
    @Enumerated(EnumType.STRING)
    private MerchantStatus status; // PENDING, APPROVED, REJECTED
    // 其他字段...
}

// 菜品实体
@Entity
@Data
public class Dish {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private BigDecimal price;
    @ManyToOne
    private Merchant merchant;
    // 其他字段...
}

// 订单实体
@Entity
@Data
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @ManyToOne
    private Merchant merchant;
    @Enumerated(EnumType.STRING)
    private OrderStatus status; // CREATED, PROCESSING, COMPLETED
    // 其他字段...
}
@RestController
@RequestMapping("/api/merchants")
public class MerchantController {
    @PostMapping("/apply")
    public ResponseEntity<?> apply(@RequestBody MerchantApplyDTO dto) {
        // 验证并保存申请
        return ResponseEntity.ok("申请已提交");
    }

    @PutMapping("/{id}/approve")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<?> approve(@PathVariable Long id) {
        // 审核逻辑
        return ResponseEntity.ok("商家已审核通过");
    }
}
@RestController
@RequestMapping("/api/dishes")
public class DishController {
    @PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<?> uploadDish(
        @RequestPart DishDTO dish,
        @RequestPart MultipartFile image) {
        // 保存菜品和图片
        return ResponseEntity.ok("菜品上传成功");
    }
}
<template>
  <form @submit.prevent="submitApplication">
    <input v-model="form.name" placeholder="商家名称" required>
    <input v-model="form.contact" placeholder="联系方式" required>
    <button type="submit">提交申请</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      form: { name: '', contact: '' }
    }
  },
  methods: {
    async submitApplication() {
      await this.$axios.post('/api/merchants/apply', this.form);
      alert('申请已提交！');
    }
  }
}
</script>
<template>
  <select v-model="selectedStatus" @change="updateStatus">
    <option value="PROCESSING">处理中</option>
    <option value="COMPLETED">已完成</option>
  </select>
</template>

<script>
export default {
  props: ['orderId'],
  data() {
    return { selectedStatus: '' }
  },
  methods: {
    async updateStatus() {
      await this.$axios.put(`/api/orders/${this.orderId}/status`, {
        status: this.selectedStatus
      });
    }
  }
}
</script>
