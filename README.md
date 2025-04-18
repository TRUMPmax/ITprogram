// User.java
@Entity
@Data
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String username;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(name = "password_hash", nullable = false)
    private String password;
    
    private String phone;
    
    @Enumerated(EnumType.STRING)
    private UserStatus status;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<VerificationCode> verificationCodes;
}

// VerificationCode.java
@Entity
@Data
public class VerificationCode {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    private String code;
    
    @Enumerated(EnumType.STRING)
    private CodeType type;
    
    private LocalDateTime expiresAt;
}
// UserService.java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;
    
    @Transactional
    public User register(RegisterDTO dto) {
        if (userRepository.existsByUsername(dto.getUsername())) {
            throw new BusinessException("用户名已存在");
        }
        
        User user = new User();
        user.setUsername(dto.getUsername());
        user.setEmail(dto.getEmail());
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        user.setPhone(dto.getPhone());
        user.setStatus(UserStatus.UNVERIFIED);
        
        User savedUser = userRepository.save(user);
        
        // 发送验证码
        String code = generateVerificationCode();
        emailService.sendVerificationEmail(user.getEmail(), code);
        
        return savedUser;
    }
    
    private String generateVerificationCode() {
        return String.valueOf(new Random().nextInt(899999) + 100000);
    }
}
