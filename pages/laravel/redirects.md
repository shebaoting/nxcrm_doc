---
title: HTTP 重定向
---


## 创建重定向

重定向响应是 `Illuminate\Http\RedirectResponse` 类的实例，并包含将用户重定向到另一个 URL 所需的正确标头。有几种方法可以生成 `RedirectResponse` 实例。最简单的方法是使用全局 `redirect` 辅助函数：

    Route::get('/dashboard', function () {
        return redirect('/home/dashboard');
    });

有时您可能希望将用户重定向到他们之前的位置，例如当提交的表单无效时。您可以使用全局 `back` 辅助函数来实现。由于此功能利用了[会话](/docs/{{version}}/session)，请确保调用 `back` 函数的路由正在使用 `web` 中间件组或已应用所有会话中间件：

    Route::post('/user/profile', function () {
        // 验证请求...

        return back()->withInput();
    }); 
## 重定向到命名路由

当您调用不带参数的 `redirect` 助手时，会返回一个 `Illuminate\Routing\Redirector` 的实例，允许您在 `Redirector` 实例上调用任何方法。例如，要生成一个重定向到命名路由的 `RedirectResponse`，您可以使用 `route` 方法：

```php
return redirect()->route('login');
```

如果您的路由具有参数，您可以将它们作为第二个参数传递给 `route` 方法：

```php
// 对于具有以下 URI 的路由：profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

为了方便起见，Laravel 还提供了全局的 `to_route` 函数：

```php
return to_route('profile', ['id' => 1]);
```

#### 通过 Eloquent 模型填充参数

如果您要重定向到一个路由，其中的“ID”参数是从 Eloquent 模型中填充的，您可以传递模型本身。ID 将自动被提取：

```php
// 对于具有以下 URI 的路由：profile/{id}

return redirect()->route('profile', [$user]);
```

如果您想要自定义放置在路由参数中的值，您应该在您的 Eloquent 模型上重写 `getRouteKey` 方法：

```php
/**
 * 获取模型路由键的值。
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

## 重定向到控制器操作

您也可以生成重定向到[控制器操作](/docs/{{version}}/controllers)。为此，将控制器和操作名称传递给 `action` 方法：

```php
use App\Http\Controllers\HomeController;

return redirect()->action([HomeController::class, 'index']);
```

如果您的控制器路由需要参数，您可以将它们作为第二个参数传递给 `action` 方法：

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```
## 使用闪存会话数据进行重定向

将用户重定向到新的 URL 并[将会话数据闪存到会话中](/docs/{{version}}/session#flash-data)通常是同时进行的。通常，在成功执行操作后会这样做，例如将会话中的成功消息闪存到会话中。为了方便起见，您可以创建一个`RedirectResponse`实例，并在一个流畅的方法链中将会话数据闪存到会话中：

```php
Route::post('/user/profile', function () {
    // 更新用户的个人资料...

    return redirect('/dashboard')->with('status', '个人资料已更新！');
});
```

您可以使用`RedirectResponse`实例提供的`withInput`方法，在将用户重定向到新位置之前，将会话的当前请求输入数据闪存到会话中。一旦将输入闪存到会话中，您可以在下一个请求中轻松地[检索它](/docs/{{version}}/requests#retrieving-old-input)：

```php
return back()->withInput();
```

在用户被重定向后，您可以从[会话](/docs/{{version}}/session)中显示闪存消息。例如，使用[Blade 语法](/docs/{{version}}/blade)：

```html
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```