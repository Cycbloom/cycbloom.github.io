---
title: "Chapter6: 任务管理系统的设计与实现"
description: "深入解析 TactiCore 任务管理系统的设计与实现过程"
date: 2025-04-20T16:00:00+08:00
slug: hugo-to-my-blog/chapter6
tag:
  - 技术
  - React
  - TypeScript
  - Zustand
  - Material-UI
series:
  - 自主开发全栈博客系统全过程
image:
math:
license:
hidden: false
comments: true
---

---

## Chapter6: 任务管理系统的设计与实现

——基于 React 和 Zustand 的现代化任务管理系统

---

### 一、状态管理设计

TactiCore 的任务管理系统采用 Zustand 进行状态管理，实现了高效的任务数据处理和过滤功能。

#### 1. 任务状态存储设计

使用 Zustand 创建任务状态存储：

```typescript
interface TaskState {
  tasks: Task[];
  loading: boolean;
  error: string | null;
  selectedTask: Task | null;
  filters: {
    status: string;
    priority: string;
    search: string;
  };
}

interface TaskActions {
  setTasks: (tasks: Task[]) => void;
  setLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
  setSelectedTask: (task: Task | null) => void;
  setFilters: (filters: Partial<TaskState["filters"]>) => void;
  addTask: (task: Task) => void;
  updateTask: (task: Task) => void;
  deleteTask: (id: string) => void;
}

const useTaskStore = create<TaskState & TaskActions>((set) => ({
  // 初始状态
  tasks: [],
  loading: false,
  error: null,
  selectedTask: null,
  filters: {
    status: "all",
    priority: "all",
    search: "",
  },

  // 状态更新方法
  setTasks: (tasks) => set({ tasks }),
  setLoading: (loading) => set({ loading }),
  setError: (error) => set({ error }),
  setSelectedTask: (task) => set({ selectedTask: task }),
  setFilters: (filters) =>
    set((state) => ({
      filters: { ...state.filters, ...filters },
    })),
  addTask: (task) =>
    set((state) => ({
      tasks: [...state.tasks, task],
    })),
  updateTask: (task) =>
    set((state) => ({
      tasks: state.tasks.map((t) => (t.id === task.id ? task : t)),
    })),
  deleteTask: (id) =>
    set((state) => ({
      tasks: state.tasks.filter((t) => t.id !== id),
    })),
}));
```

**状态管理特点**：

- **状态集中管理**：所有任务相关状态统一管理
- **类型安全**：使用 TypeScript 接口确保类型安全
- **不可变更新**：使用不可变方式更新状态
- **过滤状态**：包含任务过滤条件的状态管理

### 二、API 层设计

#### 1. 任务 API 封装

使用 Axios 封装任务相关的 API 请求：

```typescript
export const taskApi = {
  // 获取任务列表
  getTasks: (filters: FilterFormData) => {
    const cleanedFilters = cleanFilters(filters);
    return request.get<Task[]>("/tasks", {
      params: cleanedFilters,
      headers: {
        "Content-Type": "application/json",
      } as AxiosRequestHeaders,
    });
  },

  // 获取单个任务
  getTask: (id: string) => {
    return request.get<Task>(`/tasks/${id}`);
  },

  // 创建任务
  createTask: (taskData: TaskFormData) => {
    return request.post<Task>("/tasks", taskData);
  },

  // 更新任务
  updateTask: (id: string, taskData: Partial<TaskFormData>) => {
    return request.put<Task>(`/tasks/${id}`, taskData);
  },

  // 删除任务
  deleteTask: (id: string) => {
    return request.delete(`/tasks/${id}`);
  },
};
```

**API 特点**：

- **智能过滤**：自动清理无效的过滤条件
- **类型安全**：使用泛型确保响应类型
- **统一管理**：集中管理所有任务相关的 API

### 三、组件实现

#### 1. 任务页面组件

TaskPage 作为任务管理的容器组件：

```typescript
const TaskPage: React.FC = () => {
  const {
    tasks,
    loading,
    error,
    filters,
    setTasks,
    setLoading,
    setError,
    setFilters,
    addTask,
    updateTask,
    deleteTask,
  } = useTaskStore();

  useEffect(() => {
    const fetchTasks = async () => {
      try {
        setLoading(true);
        const data = await taskApi.getTasks(filters as FilterFormData);
        setTasks(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : "获取任务失败");
      } finally {
        setLoading(false);
      }
    };

    fetchTasks();
  }, [filters]);

  // ... 任务操作处理方法

  return (
    <Container maxWidth="lg">
      <Box sx={{ my: 4 }}>
        <Typography variant="h4" component="h1" gutterBottom>
          任务管理
        </Typography>
        <TaskList
          tasks={tasks}
          filters={filters as FilterFormData}
          onFilterChange={handleFilterChange}
          onEditTask={handleEditTask}
          onDeleteTask={handleDeleteTask}
          onToggleStatus={handleToggleStatus}
          onCreateTask={handleCreateTask}
        />
      </Box>
    </Container>
  );
};
```

#### 2. 任务列表组件

TaskList 负责展示任务列表和过滤功能：

```typescript
const TaskList: React.FC<TaskListProps> = ({
  tasks,
  filters,
  onFilterChange,
  onEditTask,
  onDeleteTask,
  onToggleStatus,
  onCreateTask,
}) => {
  const [isCreateDialogOpen, setIsCreateDialogOpen] = useState(false);

  return (
    <Box>
      <Stack spacing={2} sx={{ mb: 3 }}>
        <TaskFilter filters={filters} onFilterChange={onFilterChange} />
        <Button variant="contained" onClick={() => setIsCreateDialogOpen(true)}>
          创建任务
        </Button>
      </Stack>

      <Stack spacing={2}>
        {tasks.map((task) => (
          <TaskCard
            key={task.id}
            task={task}
            onEdit={() => onEditTask(task)}
            onDelete={() => onDeleteTask(task.id)}
            onStatusChange={() => onToggleStatus(task.id)}
          />
        ))}
      </Stack>

      <Dialog
        open={isCreateDialogOpen}
        onClose={() => setIsCreateDialogOpen(false)}
        maxWidth="sm"
        fullWidth
      >
        <Box sx={{ p: 3 }}>
          <TaskForm
            onSubmit={onCreateTask}
            onCancel={() => setIsCreateDialogOpen(false)}
          />
        </Box>
      </Dialog>
    </Box>
  );
};
```

#### 3. 任务过滤组件

TaskFilter 实现任务过滤功能：

```typescript
const TaskFilter: React.FC<TaskFilterProps> = ({ filters, onFilterChange }) => {
  return (
    <BaseForm<FilterFormData>
      onSubmit={onFilterChange}
      defaultValues={filters}
      schema={filterSchema}
      formTitle=""
      submitButtonText=""
      resetAfterSubmit={false}
      onFormDataChange={onFilterChange}
    >
      <Box sx={{ display: "flex", gap: 2, alignItems: "flex-start" }}>
        <Box sx={{ flex: 2 }}>
          <FormInput name="search" label="搜索任务" />
        </Box>
        <Box sx={{ flex: 1 }}>
          <GenericSelect
            name="status"
            label="状态筛选"
            options={statusOptionsWithAll}
          />
        </Box>
        <Box sx={{ flex: 1 }}>
          <GenericSelect
            name="priority"
            label="优先级筛选"
            options={priorityOptionsWithAll}
          />
        </Box>
      </Box>
    </BaseForm>
  );
};
```

### 四、路由配置

使用 React Router 配置任务管理路由：

```typescript
export const router = createBrowserRouter([
  {
    path: "/",
    element: (
      <ProtectedRoute>
        <Outlet />
      </ProtectedRoute>
    ),
    children: [
      {
        path: "tasks",
        element: <TaskPage />,
      },
    ],
  },
]);
```

### 五、总结与展望

TactiCore 的任务管理系统实现了一个完整的现代化任务管理解决方案。

**技术亮点**：

- **状态管理**：使用 Zustand 实现高效的状态管理
- **组件设计**：采用组件化设计，提高代码复用性
- **类型安全**：全面使用 TypeScript 确保类型安全
- **实时过滤**：实现了实时任务过滤功能
- **响应式设计**：使用 Material-UI 实现响应式界面

**未来展望**：

1. 添加任务拖拽排序功能
2. 实现任务标签系统
3. 添加任务统计和分析功能
4. 优化任务搜索性能
5. 添加任务导出功能

通过这些模块的实现，TactiCore 提供了一个功能完整、用户友好的任务管理系统，为用户提供高效的任务管理体验。
