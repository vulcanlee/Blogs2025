# Github Copilot 9 : 修改使用ActionFilter減少重複程式

>- 驗證邏輯重複、可抽共用  
>  - `Create` 與 `Update` 皆有 ModelState 驗證與錯誤字串組合，可考慮>抽成私有方法或 `ActionFilter`，減少重複程式。

這裡是原有關於 Create 與 Update 的方法程式碼

```csharp
[HttpPost]
public async Task<ActionResult<ApiResult<ProjectDto>>> Create([FromBody] ProjectCreateUpdateDto projectDto)
{
    try
    {
        if (!ModelState.IsValid)
        {
            var errors = string.Join("; ", ModelState.Values
                .SelectMany(v => v.Errors)
                .Select(e => e.ErrorMessage));
            return BadRequest(ApiResult<ProjectDto>.ValidationError(errors));
        }

        // 檢查專案名稱是否重複
        if (await projectRepository.ExistsByNameAsync(projectDto.Name))
        {
            return Conflict(ApiResult<ProjectDto>.ConflictResult($"專案名稱 '{projectDto.Name}' 已存在"));
        }

        // DTO 轉 Entity
        var project = mapper.Map<Project>(projectDto);
        var createdProject = await projectRepository.AddAsync(project);

        // Entity 轉 DTO
        var createdProjectDto = mapper.Map<ProjectDto>(createdProject);
        return Ok(ApiResult<ProjectDto>.SuccessResult(createdProjectDto, "新增專案成功"));
    }
    catch (Exception ex)
    {
        logger.LogError(ex, "新增專案時發生錯誤");
        return StatusCode(500, ApiResult<ProjectDto>.ServerErrorResult("新增專案時發生錯誤", ex.Message));
    }
}

[HttpPut("{id}")]
public async Task<ActionResult<ApiResult>> Update(int id, [FromBody] ProjectCreateUpdateDto projectDto)
{
    try
    {
        if (!ModelState.IsValid)
        {
            var errors = string.Join("; ", ModelState.Values
                .SelectMany(v => v.Errors)
                .Select(e => e.ErrorMessage));
            return BadRequest(ApiResult.ValidationError(errors));
        }

        if (id != projectDto.Id)
        {
            return BadRequest(ApiResult.ValidationError("路由 ID 與專案 ID 不符"));
        }

        // 檢查專案名稱是否與其他專案重複
        if (await projectRepository.ExistsByNameAsync(projectDto.Name, id))
        {
            return Conflict(ApiResult.ConflictResult($"專案名稱 '{projectDto.Name}' 已被其他專案使用"));
        }

        // DTO 轉 Entity
        var project = mapper.Map<Project>(projectDto);
        var success = await projectRepository.UpdateAsync(project);

        if (!success)
        {
            return NotFound(ApiResult.NotFoundResult($"找不到 ID 為 {id} 的專案"));
        }

        return Ok(ApiResult.SuccessResult("更新專案成功"));
    }
    catch (Exception ex)
    {
        logger.LogError(ex, "更新專案 ID {Id} 時發生錯誤", id);
        return StatusCode(500, ApiResult.ServerErrorResult("更新專案時發生錯誤", ex.Message));
    }
}
```

![](../Images/cs2025-942.png)


```
對於 Create 與 Update 方法，皆有 ModelState 驗證與錯誤字串組合，修改使用ActionFilter減少重複程式
```

![](../Images/cs2025-941.png)


![](../Images/cs2025-940.png)

* 其中提到的參考中兩個檔案，分別是 [ProjectController.cs] 與 [APIResult.cs]。

![](../Images/cs2025-939.png)

* 現在點選關於 [ApiValidationFilterAttribute] 的 [套用] 按鈕，讓 Copilot 自動幫我們修改程式碼。

![](../Images/cs2025-938.png)

* 接著在 [ApiValidationFilterAttribute] 視窗中，按下 [Tab] 按鍵，讓 Copilot 自動幫我們產生程式碼。

* 此時，在方案總管視窗內，將會看到 [ApiValidationFilterAttribute.cs] 目錄與檔案產生出來了

![](../Images/cs2025-937.png)

* 底下是這個程式碼檔案的內容

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;
using ProjectAssistant.Dto.Commons;

namespace ProjectAssistant.Web.Filters;

public class ApiValidationFilterAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        if (context.Controller is not ControllerBase controllerBase)
        {
            base.OnActionExecuting(context);
            return;
        }

        var modelState = controllerBase.ModelState;
        if (modelState.IsValid)
        {
            base.OnActionExecuting(context);
            return;
        }

        var errors = string.Join("; ", modelState.Values
            .SelectMany(v => v.Errors)
            .Select(e => e.ErrorMessage));

        // 根據回傳型別決定 ApiResult 泛型
        // 這裡假設 Create 回傳 ApiResult<ProjectDto>、Update 回傳 ApiResult
        var returnType = (context.ActionDescriptor as Microsoft.AspNetCore.Mvc.Controllers.ControllerActionDescriptor)?
            .MethodInfo.ReturnType;

        if (returnType?.IsGenericType == true &&
            returnType.GetGenericTypeDefinition() == typeof(ActionResult<>))
        {
            var innerType = returnType.GetGenericArguments()[0];
            if (innerType.IsGenericType &&
                innerType.GetGenericTypeDefinition() == typeof(ApiResult<>))
            {
                var apiResultGenericType = innerType.GetGenericArguments()[0];
                var genericMethod = typeof(ApiValidationFilterAttribute)
                    .GetMethod(nameof(CreateGenericValidationResult), System.Reflection.BindingFlags.NonPublic | System.Reflection.BindingFlags.Static)!
                    .MakeGenericMethod(apiResultGenericType);

                var result = (IActionResult)genericMethod.Invoke(null, new object[] { errors })!;
                context.Result = result;
                return;
            }
        }

        // 非泛型 ApiResult 或推斷失敗時走這裡
        context.Result = new BadRequestObjectResult(ApiResult.ValidationError(errors));
    }

    private static IActionResult CreateGenericValidationResult<T>(string errors)
    {
        return new BadRequestObjectResult(ApiResult<T>.ValidationError(errors));
    }
}
```

* 繼續回到 [Github Copilot 聊天] 視窗中
* 往下捲動到第二點，關於 在 [Program.cs] 全域註冊 Filter 的區域

![](../Images/cs2025-936.png)
* 點選 [Program.cs] 的 [套用] 按鈕，讓 Copilot 自動幫我們修改程式碼
* 在 Visual Studio 內，會自動開啟 [Program.cs] 檔案，並且將游標定位在剛剛修改的程式碼區域

![](../Images/cs2025-935.png)
* 這裡似乎是 Github Copilot 發生問題，他要自動套用的程式碼與說明的不相同。
* 因此，點選 [複製程式碼區塊] 按鈕，將程式碼複製起來
* 然後手動貼到 [Program.cs] 的適當位置

```csharp
builder.Services.AddControllers(options =>
{
    options.Filters.Add<ApiValidationFilterAttribute>();
});
```


