# Github Copilot 8 : 剖析Web API的程式碼並給出建議

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProjectController : ControllerBase
{
    private readonly ILogger<ProjectController> logger;
    private readonly ProjectRepository projectRepository;
    private readonly IMapper mapper;

    public ProjectController(ILogger<ProjectController> logger,
        ProjectRepository projectRepository,
        IMapper mapper)
    {
        this.logger = logger;
        this.projectRepository = projectRepository;
        this.mapper = mapper;
    }

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
}
```


```csharp
public class ProjectRepository
{
    private readonly BackendDBContext context;

    public ProjectRepository(BackendDBContext context)
    {
        this.context = context;
    }

    public async Task<Project> AddAsync(Project project)
    {
        project.CreatedAt = DateTime.Now;
        project.UpdatedAt = DateTime.Now;
        project.GanttChart = null; // 新增專案時不建立甘特圖   

        await context.Project.AddAsync(project);
        await context.SaveChangesAsync();

        return project;
    }
}
```



