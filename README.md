@RestController
@RequestMapping("/api")
public class LoginController {
    
    private final UserService userService;
    private final JwtTokenProvider jwtTokenProvider;

    @Autowired
    public LoginController(UserService userService, JwtTokenProvider jwtTokenProvider) {
        this.userService = userService;
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest request) {
        try {
            // 调用业务逻辑层验证用户
            User user = userService.authenticate(request.getUsername(), request.getPassword());
            
            // 生成JWT令牌
            String token = jwtTokenProvider.generateToken(user);
            
            // 返回响应
            return ResponseEntity.ok(new LoginResponse(
                token, 
                user.getId(), 
                user.getUsername(), 
                user.getRole()
            ));
            
        } catch (AuthenticationException ex) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                    .body(new ErrorResponse("用户名或密码错误"));
        }
    }
}

// DTO类
@Data
class LoginRequest {
    private String username;
    private String password;
}

@Data
@AllArgsConstructor
class LoginResponse {
    private String token;
    private Long userId;
    private String username;
    private String role;
}

@Data
@AllArgsConstructor
class ErrorResponse {
    private String message;
}
@Service
@Transactional
public class UserServiceImpl implements UserService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    @Autowired
    public UserServiceImpl(UserRepository userRepository, PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public User authenticate(String username, String password) {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("用户不存在"));
        
        if (!passwordEncoder.matches(password, user.getPassword())) {
            throw new BadCredentialsException("密码错误");
        }
        
        return user;
    }
}
