### `IEnumerable<Movie>`
代码中的`public IEnumerable<Movie> GetAllProducts()`
- 表示这个方法返回一个电影集合
- 调用方可以遍历这些电影，但不能直接修改集合（除非强制转换）
``` csharp
public IEnumerable<Product> GetAllProducts()
{
    return products;
}
```
**与具体集合类型的区别**
``` csharp
public List<Movie> GetAllMovies()    // 返回具体列表（暴露过多功能）
public Movie[] GetAllMovies()        // 返回数组（固定大小）
public IEnumerable<Movie> GetAllMovies()  // 最佳实践（仅暴露枚举能力）
```

#### 测试数据

```csharp
public class Movie
{
    public int Id { get; set; }
    public string Title { get; set; }
    public int Year { get; set; }
}

private List<Movie> _movies = new List<Movie>
{
    new Movie { Id = 1, Title = "Inception", Year = 2010 },
    new Movie { Id = 2, Title = "The Matrix", Year = 1999 },
    new Movie { Id = 3, Title = "Interstellar", Year = 2014 }
};
```

#### 1. 返回 `List<Movie>`

```csharp
public List<Movie> GetMoviesAsList()
{
    return _movies;
}

// 调用方可以：
var list = GetMoviesAsList();
list.Add(new Movie());  // 可以直接修改原始集合
list.RemoveAt(0);       // 可以删除元素
list.Clear();           // 可以清空集合
```

**问题**：暴露了太多集合修改功能，可能意外修改内部数据

#### 2. 返回 `Movie[]`

```csharp
public Movie[] GetMoviesAsArray()
{
    return _movies.ToArray();
}

// 调用方可以：
var array = GetMoviesAsArray();
array[0] = new Movie();  // 可以修改数组元素
// array.Add() 不可用，但可以修改现有元素
```

**问题**：数组长度固定，但仍暴露了元素修改能力

#### 3. 返回 `IEnumerable<Movie>`

```csharp
public IEnumerable<Movie> GetMoviesAsEnumerable()
{
    return _movies.AsReadOnly(); // 或直接 return _movies;
}

// 调用方只能：
var enumerable = GetMoviesAsEnumerable();
foreach (var m in enumerable) { ... }  // 只能遍历
// enumerable.Add() 不可用
// enumerable[0] 不可直接访问
```

**优势**：最安全，只暴露最小必要功能

#### Web API 中的实际表现

当这三种方法作为API端点时：

```csharp
// Web API 控制器
[HttpGet("list")]
public List<Movie> GetAsList() => GetMoviesAsList();

[HttpGet("array")]
public Movie[] GetAsArray() => GetMoviesAsArray();

[HttpGet("enumerable")]
public IEnumerable<Movie> GetAsEnumerable() => GetMoviesAsEnumerable();
```

**HTTP响应结果完全相同**：
```json
// 三种方法返回的JSON结果没有区别
[
  { "id": 1, "title": "Inception", "year": 2010 },
  { "id": 2, "title": "The Matrix", "year": 1999 },
  { "id": 3, "title": "Interstellar", "year": 2014 }
]
```
