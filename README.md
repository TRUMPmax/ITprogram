@RestController @RequestMapping("/api/auth") public class AuthController { private final AuthService authService;

public AuthController(AuthService authService) {
    this.authService = authService;
}

@PostMapping("/register")
public ResponseEntity<?> register(@RequestBody UserRegistrationDto dto) {
    User user = authService.registerUser(dto);
    return ResponseEntity.created(URI.create("/users/" + user.getId())).build();
}

@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    // Spring Security处理登录逻辑
    return ResponseEntity.ok().build();
}

@PostMapping("/reset-password")
public ResponseEntity<?> resetPassword(@RequestBody PasswordResetRequest request) {
    authService.initiatePasswordReset(request.getEmail());
    return ResponseEntity.accepted().build();
}
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
