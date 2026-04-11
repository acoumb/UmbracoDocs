---
description: >-
  Configure basic authentication settings for protecting the Umbraco
  website with backoffice credentials.
---

# Basic Authentication Settings

Basic authentication protects the front-end of your Umbraco website using backoffice user credentials. When enabled, visitors must authenticate before accessing any page.

A basic authentication section with all default values:

{% code title="appsettings.json" %}
```json
"Umbraco": {
  "CMS": {
    "BasicAuth": {
      "AllowedIPs": [],
      "Enabled": false,
      "RedirectToLoginPage": false,
      "SharedSecret": {
        "HeaderName": "X-Authentication-Shared-Secret",
        "Value": null
      },
      "LoginViewPath": "/umbraco/BasicAuthLogin/Login.cshtml",
      "TwoFactorViewPath": "/umbraco/BasicAuthLogin/TwoFactor.cshtml"
    }
  }
}
```
{% endcode %}

## AllowedIPs

A comma-separated list of IP addresses to limit where requests can come from.

## Enabled

Enables or disables basic authentication. The default value is `false`.

## RedirectToLoginPage

When set to `true`, users are redirected to a standalone server-rendered login page at `/umbraco/basic-auth/login` instead of seeing the browser's native authentication popup. The default value is `false`.

This setting is required for [External login providers](../security/external-login-providers.md) and [Two-factor Authentication](../security/two-factor-authentication.md) to work with basic authentication.

{% hint style="info" %}
When two-factor authentication is required for a user, the login flow redirects to the 2FA page automatically. This happens even when `RedirectToLoginPage` is set to `false`, because the browser's native popup cannot complete a 2FA flow.
{% endhint %}

## SharedSecret

A shared secret sent via an HTTP header to bypass basic authentication. This is useful for server-to-server communication.

### HeaderName

The header name used to compare the shared secret. The default value is `X-Authentication-Shared-Secret`.

### Value

The value of the shared secret. Must be a non-empty string to be enabled. The default value is `null`.

## LoginViewPath

Path to a custom Razor view for the login page. When omitted, the built-in login view is used.

See [Customizing the login views](basicauthsettings.md#customizing-the-login-views) for details.

## TwoFactorViewPath

Path to a custom Razor view for the two-factor authentication (2FA) page. When omitted, the built-in 2FA view is used.

See [Customizing the login views](basicauthsettings.md#customizing-the-login-views) for details.

## Login flow

When `RedirectToLoginPage` is set to `true`, the login flow works as follows:

1. A visitor requests a protected page.
2. The middleware redirects to `/umbraco/basic-auth/login?returnPath=...`.
3. The visitor enters their backoffice credentials.
4. If two-factor authentication is required, the visitor is redirected to `/umbraco/basic-auth/2fa`.
5. On successful authentication, the visitor is redirected back to the original page.

External login providers appear as buttons on the login page when configured. See the [External login providers](../security/external-login-providers.md) article for setup instructions.

## Frontend-only deployments

Basic authentication works in frontend-only deployments where the backoffice is not available. To enable this, register `AddBackOfficeSignIn()` in your `Program.cs`:

{% code title="Program.cs" %}
```csharp
builder.CreateUmbracoBuilder()
    .AddCore()
    .AddBackOfficeSignIn()
    .AddWebsite()
    .AddComposers()
    .Build();
```
{% endcode %}

`AddBackOfficeSignIn()` registers backoffice identity and cookie authentication without the full backoffice. For details on the available configurations, see the [Service Registration](../service-registration.md) article.

{% hint style="info" %}
When using the full backoffice setup with `AddBackOffice()`, backoffice sign-in is included automatically. You do not need to add `AddBackOfficeSignIn()` separately.
{% endhint %}

## Customizing the login views

The built-in login and 2FA pages use minimal styling and work without customization. To match your site's design, you can provide custom Razor views.

Set the view paths in `appsettings.json`:

{% code title="appsettings.json" %}
```json
"Umbraco": {
  "CMS": {
    "BasicAuth": {
      "Enabled": true,
      "RedirectToLoginPage": true,
      "LoginViewPath": "~/Views/BasicAuth/Login.cshtml",
      "TwoFactorViewPath": "~/Views/BasicAuth/TwoFactor.cshtml"
    }
  }
}
```
{% endcode %}

The login view receives a `BasicAuthLoginModel` with the following properties:

- `ReturnPath` — the URL to redirect to after login.
- `ErrorMessage` — an error message to display (null when no error).
- `ExternalLoginProviders` — a list of configured external login providers to render as buttons.

The 2FA view receives a `BasicAuthTwoFactorModel` with the following properties:

- `ReturnPath` — the URL to redirect to after verification.
- `ErrorMessage` — an error message to display (null when no error).
- `TwoFactorProviders` — a list of available 2FA providers.

{% hint style="info" %}
Use the built-in views at `/umbraco/BasicAuthLogin/Login.cshtml` and `/umbraco/BasicAuthLogin/TwoFactor.cshtml` as a reference when creating custom views.
{% endhint %}
