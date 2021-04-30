# JWTRule

用于认证的 JSON Web Token（JWT）令牌格式，由 RFC 7519 定义。参见 OAuth 2.0 和 OIDC 1.0，了解在整个认证流程中如何使用。

## 示例

JWT 的规格是由 `https://example.com` 签发，受众要求必须是 `bookstore_android.apps.example.com` 或 `bookstore_web.apps.example.com`。该令牌应呈现在 `Authorization` header（默认）。Json 网络密钥集（JWKS）将按照 OpenID Connect 协议被发现。

```yaml
issuer: https://example.com
audiences:
- bookstore_android.apps.example.com
  bookstore_web.apps.example.com
```

这个例子在非默认位置（`x-goog-iap-jwt-assertion` header）指定了令牌。它还定义了 URI 来明确获取 JWKS。

```yaml
issuer: https://example.com
jwksUri: https://example.com/.secret/jwks.json
jwtHeaders:
- "x-goog-iap-jwt-assertion"
```

关于 `JWTRule` 配置的详细用法请参考 [Istio 官方文档](https://istio.io/latest/docs/reference/config/security/request_authentication/)。

# 参考

- [JWTRule - istio.io](https://istio.io/latest/docs/reference/config/security/jwt/)

