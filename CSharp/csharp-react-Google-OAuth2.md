# React 使用 Google OAuth 2.0

在這篇文章中，我們將會建立一個 React 應用程式，並且使用 Google OAuth 2.0 來實現使用者的登入功能。這樣使用者就可以使用他們的 Google 帳號來登入我們的應用程式，並且在登入後看到一些受保護的內容。

## 建立 Blazor 專案
* 開啟 Visual Studio 2026
* 選擇「建立新專案」
* 在 [建立新專案] 視窗中，在右方清單內，找到並選擇「ASP.NET Core Web API」 項目
  >此專案範本可用於使用 ASP.NET Core 控制器或最小 API 建立 RESTful Web API，並可選擇地支援 OpenAPI和驗證
* 然後點擊右下方「下一步」按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 [WebApiDemo]
* 然後點擊右下方「下一步」按鈕
* 接著會看到 [其他資訊] 對話窗
* 在這個對話窗內，確認使用底下的選項
    * 架構：.NET 10.0 (或更新版本)
    * 驗證類型：無
    * 勾選 針對 HTTPS 進行設定
    * 啟用 OpenAPI 支援
    * 勾選 不要使用最上層陳述式 (這是我的個人習慣)
    * 使用控制器
    * 不要勾選 在 .NET Aspire 協調流程中登錄
* 然後點擊右下方「建立」按鈕
* 現在，已經完成了這個 ASP.NET Core Web API 專案的建立

## 新增設定服務

* 滑鼠右擊專案的根目錄
* 選擇 [加入] > [新增資料夾]
* 將資料夾命名為 [Services]
* 在 [Services] 資料夾上，點擊滑鼠右鍵，選擇 [加入] > [新增項目]
* 在彈出的對話窗中，選擇 [類別] 項目，將類別命名為 [ConfigurationService.cs]，然後點擊「新增」按鈕
  >這裡將會使用 [顯示精簡檢視] 的方式來選擇類別項目，這樣就不會被其他的類別項目給干擾了
* 在 [ConfigurationService.cs] 的程式碼區塊內，輸入底下的程式碼：

```csharp
namespace csBlazorGoogleOAuth2.Services;

public class ConfigurationService
{
    public string ClientId { get; set; } = "apps.googleusercontent.com";
    public string ClientSecret { get; set; } = "";
}
```

這裡的服務將會提供兩個字串，分別用於存儲 Google OAuth 2.0 的 Client ID 和 Client Secret，這些值是從 Google API Console 中獲取的，並且在後續的程式碼中會使用這些值來設定 Google 驗證的相關選項。

## 修正 OAuth2 需要用到的程式碼
* 從專案跟目錄下，找到並打開 [Program.cs] 檔案
* 在 [Program.cs] 的程式碼區塊內，使用底下程式碼來取代原本的程式碼：

```csharp

using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.Google;
using Microsoft.AspNetCore.Authorization;
using System;
using System.Security.Claims;
using WebApiDemo.Services;

namespace WebApiDemo;

public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        builder.Services.AddControllers();
        builder.Services.AddOpenApi();

        builder.Services.AddSingleton<Services.ConfigurationService>();
        builder.Services.AddCors(options =>
        {
            options.AddPolicy("AllowReactApp", policy =>
            {
                policy.WithOrigins("http://localhost:49158") // React 應用運行在此端口
                      .AllowAnyHeader()
                      .AllowAnyMethod()
                      .AllowCredentials();
            });
        });
        var configService = builder.Services.BuildServiceProvider().GetRequiredService<ConfigurationService>();

        builder.Services
            .AddAuthentication(options =>
            {
                options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
                options.DefaultChallengeScheme = GoogleDefaults.AuthenticationScheme;
            })
            .AddCookie(options =>
            {
                options.Cookie.Name = "bff_auth";
                options.Cookie.HttpOnly = true;
                options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
                options.Cookie.SameSite = SameSiteMode.Lax; // 若跨網域回跳需要，改 None 並全程 HTTPS
                options.SlidingExpiration = true;
                options.ExpireTimeSpan = TimeSpan.FromHours(8);
                options.LoginPath = "/login";
                options.LogoutPath = "/logout";
                // 重要：讓外部登入流程可保存 tokens (可選)
                options.Events = new CookieAuthenticationEvents
                {
                    OnRedirectToLogin = ctx =>
                    {
                        // 對 API 請求返回 401 而不是 302
                        if (ctx.Request.Path.StartsWithSegments("/api"))
                        {
                            ctx.Response.StatusCode = StatusCodes.Status401Unauthorized;
                            return Task.CompletedTask;
                        }
                        ctx.Response.Redirect(ctx.RedirectUri);
                        return Task.CompletedTask;
                    }
                };
            })
            .AddGoogle(GoogleDefaults.AuthenticationScheme, options =>
            {
                options.ClientId = configService.ClientId;
                options.ClientSecret = configService.ClientSecret;
                options.CallbackPath = "/signin-google"; // 要和 Google 控制台一致
                options.SaveTokens = true;               // 如需後續調用 Google API 可保留
            });

        builder.Services.AddAuthorization();

        var app = builder.Build();

        if (app.Environment.IsDevelopment())
        {
            app.MapOpenApi();
        }

        app.UseHttpsRedirection();
        app.UseCors("AllowReactApp");

        app.UseAuthentication();
        app.UseAuthorization();

        app.MapGet("/do-login", async ctx =>
        {
            var defaultReturn = "http://localhost:49158/"; // 未帶 returnUrl 時的預設回跳
            var props = new Microsoft.AspNetCore.Authentication.AuthenticationProperties
            {
                RedirectUri = "/login"
            };
            await ctx.ChallengeAsync(GoogleDefaults.AuthenticationScheme, props);
        });

        app.MapGet("/login", async ctx =>
        {
            var authResult = await ctx.AuthenticateAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            if (!authResult.Succeeded || authResult.Principal == null)
            {
                ctx.Response.StatusCode = StatusCodes.Status401Unauthorized;
                await ctx.Response.WriteAsync("Authentication failed.");
                return;
            }
            ctx.User = authResult.Principal;
            var id = ctx.User.FindFirstValue(ClaimTypes.NameIdentifier) ?? "";
            var name = ctx.User.FindFirstValue(ClaimTypes.Name) ?? "";
            var givenName = ctx.User.FindFirstValue(ClaimTypes.GivenName) ?? "";
            var surname = ctx.User.FindFirstValue(ClaimTypes.Surname) ?? "";
            var email = ctx.User.FindFirstValue(ClaimTypes.Email)
                             ?? ctx.User.Claims.FirstOrDefault(c =>
                                    c.Type.EndsWith("/email", StringComparison.OrdinalIgnoreCase) ||
                                    c.Type.EndsWith("/emailaddress", StringComparison.OrdinalIgnoreCase))?.Value
                             ?? "";

            var accessToken = authResult.Properties?.GetTokenValue("access_token");
            var refreshToken = authResult.Properties?.GetTokenValue("refresh_token");
            var expiresAt = authResult.Properties?.GetTokenValue("expires_at");
        });

        app.MapPost("/logout", async ctx =>
        {
            await ctx.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            ctx.Response.StatusCode = StatusCodes.Status204NoContent;
        });

        app.MapGet("/api/me", (HttpContext ctx) =>
        {
            if (ctx.User?.Identity?.IsAuthenticated == true)
            {
                var name = ctx.User.Identity?.Name ?? "";
                var email = ctx.User.Claims.FirstOrDefault(c => c.Type.Contains("email"))?.Value ?? "";
                return Results.Ok(new { isAuthenticated = true, name, email });
            }
            return Results.Ok(new { isAuthenticated = false });
        });

        app.MapGet("/api/secure/data", [Authorize] () =>
        {
            return Results.Ok(new
            {
                secret = "只有登入的人看得到的資料",
                at = DateTimeOffset.UtcNow
            });
        });
        #endregion

        app.MapControllers();

        app.Run();
    }
}
```

* 在這裡使用了 `builder.Services.AddSingleton<Services.ConfigurationService>();` 敘述來將 [ConfigurationService] 類別註冊為單例服務，這樣在整個應用程式的生命週期內都會使用同一個實例。

由於這個範例會由 ASP.NET Core Web API 設計後端，並且由 React 前端來呼叫，所以需要設定 CORS 來允許來自 React 應用的跨域請求。這裡使用了 `builder.Services.AddCors()` 方法來添加 CORS 服務，並且定義了一個名為 "AllowReactApp" 的 CORS 政策，該政策允許來自 `http://localhost:49158` 的請求，並且允許任何標頭、任何方法以及攜帶憑證（如 Cookie）。接著，在中介軟體管道中使用 `app.UseCors("AllowReactApp");` 來啟用這個 CORS 政策。

接著，使用了 `builder.Services.AddAuthentication()` 方法來設定應用程式的驗證機制。這裡指定了預設的驗證方案為 Cookie 驗證，並且將預設的挑戰方案設定為 Google 驗證，這意味著當需要進行驗證挑戰時，系統會自動使用 Google 驗證方案。另外，使用了 `AddCookie()` 方法來設定 Cookie 驗證的相關選項，在此使用了 [Name] 代表了驗證 Cookie 的名稱，使用了 [HttpOnly] 來指定該 Cookie 是否只能由伺服器存取，使用了 [SecurePolicy] 來指定該 Cookie 是否只能在 HTTPS 連線中傳輸，使用了 [SameSite] 來指定該 Cookie 的 SameSite 屬性，使用了 [SlidingExpiration] 和 [ExpireTimeSpan] 來設定 Cookie 的過期時間和滑動過期的行為，使用了 [LoginPath] 和 [LogoutPath] 來指定登入和登出的 URL 路徑，這裡使用的值分別為 "/login" 和 "/logout"，這些路徑在後續的程式碼中會對應到相應的端點。

另外，在 [Events] 指定給新建立的 [CookieAuthenticationEvents] 類別，其中對於該類別的屬性 `OnRedirectToLogin` 事件，我們定義了一個 lambda 表達式來處理當需要重定向到登入頁面時的行為。在這個事件處理器中，我們首先檢查當前的請求路徑是否以 "/api" 開頭，如果是的話，表示這是一個 API 請求，這時候我們不希望進行重定向，而是直接返回一個 401 Unauthorized 的狀態碼，這樣前端應用就可以根據這個狀態碼來判斷使用者是否需要登入。相反地，如果不是 API 請求，那麼我們就按照預設的行為進行重定向，將使用者導向到登入頁面。

然後，使用了 `AddGoogle()` 方法來設定 Google 驗證的相關選項，包括 [ClientId] 和 [ClientSecret]，這些值是從 Google API Console 中獲取的，並且存儲在 [ConfigurationService] 中。另外，也設定了 [CallbackPath]，這個值需要和 Google API Console 中設定的回調 URL 一致，這樣在 Google 驗證完成後，系統才能正確地處理回調請求。最後，使用了 [SaveTokens] 來指定是否將從 Google 獲取的訪問令牌和刷新令牌保存在驗證屬性中，這樣在後續的程式碼中就可以使用這些令牌來調用 Google API。

對於 [builder.Services.AddAuthorization();]，這行程式碼的作用是將授權服務添加到應用程式的服務容器中。這樣做的目的是為了在應用程式中啟用授權功能，讓我們可以使用 [Authorize] 屬性來保護特定的頁面或 API 端點，只有經過授權的使用者才能訪問這些受保護的資源。

以上是針對這個 ASP.NET Core Web API 專案的相關服務註冊所做的事

在 [var app = builder.Build();] 之後的程式碼主要是設定應用程式的中介軟體管道和路由。

找到這個敘述 [app.MapStaticAssets();] 之後，加入了 `app.UseAuthentication();` & `app.UseAuthorization();` 這兩行程式碼的作用是將驗證和授權中介軟體添加到應用程式的請求處理管道中。這樣做的目的是為了確保在處理每個請求時，系統會先進行驗證，確認使用者的身份，然後再進行授權，確認使用者是否有權訪問特定的資源。

由於前面有宣告了 CORS 相關的服務，所以在這裡也需要使用 `app.UseCors("AllowReactApp");` 來啟用 CORS 中介軟體，這樣才能允許來自 React 應用的跨域請求。需要注意的是，CORS 中介軟體應該放在管道的早期位置，這樣才能確保它能夠正確地處理跨域請求，並且在驗證和授權之前就已經處理了 CORS 的相關邏輯。

接著加入了五個服務端點，分別是 [/do-login]、[/login]、[/logout]、[/api/me] 和 [/api/secure/data]。簡單說明這五個服務端點為：
* [/do-login]：當使用者訪問這個 URL 時，系統會發起一個驗證挑戰，使用 Google 驗證方案來進行登入流程，並且指定了一個預設的回跳 URL，這樣在登入成功後，使用者就會被重定向到該 URL。
* [/login]：當使用者訪問這個 URL 時，系統會嘗試從 Cookie 驗證方案中獲取驗證結果，如果驗證成功，則將使用者的相關資訊（如 ID、名稱、電子郵件等）從驗證結果中提取出來，並且可以在這裡進行一些額外的處理，例如存儲訪問令牌或刷新令牌等。
* [/logout]：當使用者發送 POST 請求到這個 URL 時，系統會執行 `context.SignOutAsync()` 方法來登出使用者，這裡指定了 Cookie 驗證方案，這樣就會清除與該方案相關的驗證 Cookie。登出完成後，使用 `context.Response.Redirect("/")` 方法將使用者重定向回根目錄。
* [/api/me]：當使用者訪問這個 URL 時，系統會檢查使用者是否已經驗證，如果已經驗證，則返回使用者的名稱和電子郵件等資訊；如果沒有驗證，則返回一個表示未驗證的結果。
* [/api/secure/data]：當使用者訪問這個 URL 時，系統會檢查使用者是否已經驗證，如果已經驗證，則返回一些受保護的資料；如果沒有驗證，則返回一個 401 Unauthorized 的狀態碼。

## 建立 React 專案
* 滑鼠右擊解決方案 [WebApiDemo]，選擇「加入」>「新增專案」
* 在 [加入新專案] 視窗中，在右方清單內，找到並選擇「React 個應用程式」 項目
  >請注意選擇具有底下的說明項目的專案範本
  >
  >TypeScript React 專案範本，透過執行 npx 的全域安裝來進行啟動載入
* 然後點擊右下方「下一步」按鈕
* 此時將會看到 [設定新的專案] 對話窗
* 在該對話窗的 [專案名稱] 欄位中，輸入專案名稱，例如 [reactdemo]
* 然後點擊右下方「下一步」按鈕
* 然後點擊右下方「建立」按鈕
* 現在，已經完成了這個 React 個應用程式 專案的建立
* 在方案總管內，將會看到有兩個專案建立起來
  ![](../Images/cs2025-889.png)

## 修改 React 專案的程式碼
* 在方案總管內，找到並展開 [reactdemo] 專案
* 找到並打開 [src] 資料夾內的 [App.tsx] 檔案
* 將該檔案內的程式碼全部刪除，然後將底下的程式碼貼上到該檔案內

```tsx
import { useEffect, useState } from 'react';

const API = 'https://localhost:7074';

interface UserInfo {
    isAuthenticated: boolean;
    name?: string;
    email?: string;
}

interface SecureData {
    secret: string;
    at: string;
}

function App() {
    const [me, setMe] = useState<UserInfo | null>(null);
    const [secureData, setSecureData] = useState<SecureData | null>(null);
    const [loading, setLoading] = useState<boolean>(true);
    const [error, setError] = useState<string | null>(null);

    useEffect(() => {
        if (navigator.permissions && typeof navigator.permissions.query === 'function') {
            const originalQuery = navigator.permissions.query;
            navigator.permissions.query = function (descriptor: PermissionDescriptor) {
                return originalQuery.call(navigator.permissions, descriptor);
            };
        }

        const handleError = (event: ErrorEvent) => {
            if (event.error?.message?.includes('Illegal invocation')) {
                console.warn('捕獲到 Illegal invocation 錯誤，已忽略:', event.error);
                event.preventDefault(); // 防止錯誤中斷應用
            }
        };

        window.addEventListener('error', handleError);
        return () => window.removeEventListener('error', handleError);
    }, []);

    const loadMe = async () => {
        try {
            setLoading(true);
            setError(null);
            const res = await fetch(`${API}/api/me`, {
                credentials: 'include',
                headers: {
                    'Accept': 'application/json'
                }
            });

            if (!res.ok) {
                throw new Error(`HTTP error! status: ${res.status}`);
            }

            const data = await res.json();
            setMe(data);
        } catch (err) {
            console.error('載入用戶狀態失敗:', err);
            setError('無法載入用戶狀態');
            setMe({ isAuthenticated: false });
        } finally {
            setLoading(false);
        }
    };

    useEffect(() => {
        loadMe();
    }, []);

    const login = () => {
        sessionStorage.setItem('preLoginPath', window.location.pathname);
        window.location.href = `${API}/do-login`;
    };

    const logout = async () => {
        try {
            setError(null);
            const res = await fetch(`${API}/logout`, {
                method: 'POST',
                credentials: 'include'
            });

            if (!res.ok) {
                throw new Error(`登出失敗: ${res.status}`);
            }

            setMe({ isAuthenticated: false });
            setSecureData(null);

            await loadMe();
        } catch (err) {
            console.error('登出失敗:', err);
            setError('登出失敗，請稍後再試');
        }
    };

    const callSecureApi = async () => {
        try {
            setError(null);
            const res = await fetch(`${API}/api/secure/data`, {
                credentials: 'include',
                headers: {
                    'Accept': 'application/json'
                }
            });

            if (res.status === 401) {
                setError('未登入或 Cookie 已過期，請重新登入');
                setMe({ isAuthenticated: false });
                return;
            }

            if (!res.ok) {
                throw new Error(`HTTP error! status: ${res.status}`);
            }

            const data = await res.json();
            setSecureData(data);
        } catch (err) {
            console.error('呼叫 API 失敗:', err);
            setError('無法取得受保護的資料');
        }
    };

    if (loading) {
        return (
            <div style={{ padding: 24 }}>
                <h1>React + ASP.NET Core 9 + Google OAuth2 (BFF)</h1>
                <p>載入中...</p>
            </div>
        );
    }

    return (
        <div style={{ padding: 24, fontFamily: 'Arial, sans-serif' }}>
            <h1>React + ASP.NET Core 9 + Google OAuth2 (BFF)</h1>

            {/* 錯誤訊息顯示 */}
            {error && (
                <div style={{
                    padding: 12,
                    marginBottom: 16,
                    backgroundColor: '#fee',
                    border: '1px solid #fcc',
                    borderRadius: 4,
                    color: '#c00'
                }}>
                    <strong>錯誤:</strong> {error}
                </div>
            )}

            {/* 認證操作區 */}
            <section style={{ marginBottom: 24 }}>
                <h2>認證操作</h2>
                {!me?.isAuthenticated ? (
                    <button
                        onClick={login}
                        style={{
                            padding: '10px 20px',
                            fontSize: '16px',
                            backgroundColor: '#4285f4',
                            color: 'white',
                            border: 'none',
                            borderRadius: 4,
                            cursor: 'pointer'
                        }}
                    >
                        🔐 使用 Google 登入
                    </button>
                ) : (
                    <button
                        onClick={logout}
                        style={{
                            padding: '10px 20px',
                            fontSize: '16px',
                            backgroundColor: '#dc3545',
                            color: 'white',
                            border: 'none',
                            borderRadius: 4,
                            cursor: 'pointer'
                        }}
                    >
                        🚪 登出
                    </button>
                )}
            </section>

            {/* 用戶狀態顯示 */}
            <section style={{
                marginBottom: 24,
                padding: 16,
                backgroundColor: '#f5f5f5',
                borderRadius: 4
            }}>
                <h2>目前登入狀態</h2>
                {me?.isAuthenticated ? (
                    <div>
                        <p>✅ <strong>已登入</strong></p>
                        <p><strong>姓名:</strong> {me.name}</p>
                        <p><strong>Email:</strong> {me.email}</p>
                    </div>
                ) : (
                    <p>❌ <strong>未登入</strong></p>
                )}
                <details style={{ marginTop: 12 }}>
                    <summary style={{ cursor: 'pointer' }}>查看原始資料</summary>
                    <pre style={{
                        marginTop: 8,
                        padding: 12,
                        backgroundColor: '#fff',
                        border: '1px solid #ddd',
                        borderRadius: 4,
                        overflow: 'auto'
                    }}>
                        {JSON.stringify(me, null, 2)}
                    </pre>
                </details>
            </section>

            {/* 受保護 API 存取區 */}
            <section style={{
                padding: 16,
                backgroundColor: '#f5f5f5',
                borderRadius: 4
            }}>
                <h2>受保護的 API</h2>
                <button
                    onClick={callSecureApi}
                    disabled={!me?.isAuthenticated}
                    style={{
                        padding: '10px 20px',
                        fontSize: '16px',
                        backgroundColor: me?.isAuthenticated ? '#28a745' : '#ccc',
                        color: 'white',
                        border: 'none',
                        borderRadius: 4,
                        cursor: me?.isAuthenticated ? 'pointer' : 'not-allowed'
                    }}
                >
                    📡 呼叫受保護 API
                </button>

                {!me?.isAuthenticated && (
                    <p style={{ color: '#666', marginTop: 8 }}>
                        ℹ️ 請先登入才能呼叫受保護的 API
                    </p>
                )}

                {secureData && (
                    <div style={{ marginTop: 16 }}>
                        <h3>API 回應資料:</h3>
                        <div style={{
                            padding: 12,
                            backgroundColor: '#d4edda',
                            border: '1px solid #c3e6cb',
                            borderRadius: 4
                        }}>
                            <p><strong>機密資料:</strong> {secureData.secret}</p>
                            <p><strong>時間戳記:</strong> {new Date(secureData.at).toLocaleString('zh-TW')}</p>
                        </div>
                        <details style={{ marginTop: 12 }}>
                            <summary style={{ cursor: 'pointer' }}>查看原始資料</summary>
                            <pre style={{
                                marginTop: 8,
                                padding: 12,
                                backgroundColor: '#fff',
                                border: '1px solid #ddd',
                                borderRadius: 4,
                                overflow: 'auto'
                            }}>
                                {JSON.stringify(secureData, null, 2)}
                            </pre>
                        </details>
                    </div>
                )}
            </section>
        </div>
    );
}

export default App;
```

在這個 [App.tsx] 的程式碼中，我們建立了一個 React 組件，該組件會在載入時向後端的 `/api/me` 端點發送請求，以獲取當前使用者的認證狀態和相關資訊。根據認證狀態，組件會顯示不同的內容，例如登入按鈕、使用者資訊以及呼叫受保護 API 的按鈕等。當使用者點擊登入按鈕時，會觸發 `login` 函數，該函數會將使用者重定向到後端的 `/do-login` 端點，這個端點會啟動 Google OAuth 2.0 的登入流程。當使用者成功登入後，組件會更新狀態以顯示使用者的資訊，並且允許使用者呼叫受保護的 API。當使用者點擊登出按鈕時，會觸發 `logout` 函數，該函數會向後端的 `/logout` 端點發送 POST 請求來執行登出操作，並且更新組件的狀態以反映使用者已經登出的狀態。

[API] 這個常數將會用於存儲後端 API 的基礎 URL，這樣在後續的程式碼中就可以使用這個常數來構建完整的 API 請求 URL，這樣做的好處是當後端 API 的 URL 發生變化時，只需要修改這個常數的值，而不需要在整個程式碼中搜尋和替換所有的 API URL。

[loadMe] 這個函數的作用是向後端的 `/api/me` 端點發送請求，以獲取當前使用者的認證狀態和相關資訊。該函數首先設置了載入狀態和錯誤狀態，然後使用 `fetch` API 發送 GET 請求到 `/api/me`，並且攜帶了 `credentials: 'include'` 的選項，這樣可以確保在跨域請求中攜帶 Cookie。當收到回應後，該函數會檢查回應的狀態碼，如果不是成功的狀態碼，就會拋出一個錯誤。接著，該函數會將回應解析為 JSON 格式，並且更新組件的狀態以存儲使用者的資訊。如果在請求過程中發生任何錯誤，該函數會捕獲錯誤並且更新錯誤狀態，以便在 UI 中顯示相應的錯誤訊息。

[login] 這個函數的作用是觸發使用者的登入流程。當使用者點擊登入按鈕時，該函數會將當前的路徑存儲在 `sessionStorage` 中，這樣在登入完成後可以將使用者重定向回原本的頁面。接著，該函數會將瀏覽器的地址重定向到後端的 `/do-login` 端點，這個端點會啟動 Google OAuth 2.0 的登入流程。

[logout] 這個函數的作用是觸發使用者的登出流程。當使用者點擊登出按鈕時，該函數會向後端的 `/logout` 端點發送 POST 請求來執行登出操作，並且攜帶了 `credentials: 'include'` 的選項，以確保在跨域請求中攜帶 Cookie。當收到回應後，該函數會檢查回應的狀態碼，如果不是成功的狀態碼，就會拋出一個錯誤。接著，該函數會更新組件的狀態以反映使用者已經登出的狀態，並且呼叫 `loadMe()` 函數來重新載入使用者的認證狀態。

[callSecureApi] 這個函數的作用是向後端的 `/api/secure/data` 端點發送請求，以獲取一些受保護的資料。該函數首先設置了錯誤狀態，然後使用 `fetch` API 發送 GET 請求到 `/api/secure/data`，並且攜帶了 `credentials: 'include'` 的選項，以確保在跨域請求中攜帶 Cookie。當收到回應後，該函數會檢查回應的狀態碼，如果是 401 Unauthorized，表示使用者未登入或 Cookie 已過期，這時候會更新錯誤狀態並且將使用者的認證狀態設置為未登入。如果回應的狀態碼不是成功的狀態碼，就會拋出一個錯誤。接著，該函數會將回應解析為 JSON 格式，並且更新組件的狀態以存儲從 API 獲取的受保護資料。

## 設定同時啟動多個專案
* 在這個方案內，擁有兩個專案，分別是 [WebApiDemo] 與 [reactdemo]
* 前者是 ASP.NET Core Web API 專案，後者是 React 的前端專案
* 因此，我們需要設定 Visual Studio 來同時啟動這兩個專案，才能在開發過程中同時測試前後端的功能
* 在方案總管內，右擊方案 [WebApiDemo]，選擇 [設定啟動專案]
* 在 [方案 'ReactWebApi' 屬性頁] 對話窗內，選擇 [多個啟動專案] 的選項
* 在下方的專案列表內，找到 [WebApiDemo] 與 [reactdemo] 這兩個專案
* 將這兩個專案的 [動作] 欄位都設定為 [啟動]
* 然後點擊右下方的 [確定] 按鈕，來儲存這個設定
![](../Images/cs2025-882.png)

