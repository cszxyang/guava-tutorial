
## 概述

图被用于描述实体及实体间的关系，这样的例子有如网页和超链接、科学家及其撰写的论文、机场及其之间的路线等等。`com.google.common.graph` 包提供了对 `图` 这种数据结构的实现，

从数据结构来看，图是非线性的，它由一组节点和一组边组成，每条边将两个节点相连，这两个节点亦称端点（endpoints）。根据是否区分端点为起始节点和目标节点，又可将边分为有向边和无向边。如果图的每个边都是有向的，则图是有向的；如果图的每个边都是无向的，则图是无向的。common.graph 不支持同时具有有向边和无向边的图。

<div align="center"><img src="./assets/graph-wiki.png"></img></div>

<

看下下面的例子

```java
graph.addEdge(nodeU, nodeV, edgeUV);
```

- nodeU 和 nodeV 两个节点彼此相邻
- edgeUV 是入射到 nodeU 并 nodeV（且反之亦然）

如果 graph 是有向的，则 nodeU 是 nodeV 的前继节点，nodeV 是 nodeU 的后继节点，即边 edgeUV 从节点 nodeU 指向 nodeV。如果 graph 是无向的，则 nodeU 是 nodeV 同时为彼此的前继节点和后继节点。所有这些关系都取决于 graph 的类型。

在图论中，自环（self-loop）是一条顶点与自身连接的边。如果自环是有向的，则对于环中那个节点而言，自环既是出边也是入边；对于自环而言，环中那个节点既是源节点也是目标节点，见下图中的节点4.

<div align="center"><img src="./assets/self-loop.jfif"></img></div>

如果两条边所连接的相同的节点以相同的顺序（如果有的话），则称它们是平行的。如果它们所连接的相同的节点以相反的顺序，则称它们是反平行的（无方向的边不能反平行）。看下面的示例：

```java
directedGraph.addEdge(nodeU, nodeV, edgeUV_a);
directedGraph.addEdge(nodeU, nodeV, edgeUV_b);
directedGraph.addEdge(nodeV, nodeU, edgeVU);

undirectedGraph.addEdge(nodeU, nodeV, edgeUV_a);
undirectedGraph.addEdge(nodeU, nodeV, edgeUV_b);
undirectedGraph.addEdge(nodeV, nodeU, edgeVU);
```

在 directedGraph 中，edgeUV_a 和 edgeUV_b 是彼此平行的，而它们都是与 edgeVU 反平行的。在 undirectedGraph 中，edgeUV_a、edgeUV_b 和 edgeVU 中每个都与另外两个彼此平行。

## guava 的图类库功能

`common.graph` 专注于提供接口和类以支持使用图。它不提供 I/O 或可视化支持等功能，并且工具类的选择非常有限。更多相关详见 [FAQ](https://github.com/google/guava/wiki/GraphsExplained#faq)。
 
总体而言，`common.graph` 支持以下类型的图：

- 有向图
- 无向图
- 具有关联值（权重，标签等）的节点和/或边
- 允许/不允许自循环的图
- 允许/不允许平行边的图（具有平行边的图有时称为多图）
- 节点/边按插入顺序排列、排序或无序的图

特定 `common.graph` 类型支持的图的类型在其 Javadoc 中有说明。Javadoc 在其相关 Builder 类型中指定了每种图形类型的内置实现所支持的图类型。该库中类型的特定实现（尤其是第三方实现）不需要支持所有这些变体，并且可能还支持其他变体。

该库与基础数据结构的选择无关：可以将关系存储为矩阵、邻接表、邻接图等，具体取决于实现者想要优化的用例。

`common.graph`（目前）不提供对以下图变体的明确支持，尽管可以使用现有的类型对它们进行建模：

- 树木，森林
- 具有相同类型（节点或边）的元素且具有不同类型的图（例如：二分图/k分图，多峰图）
- 超图

 [`Graphs`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Graphs.html) 不允许图形同时具有有向边和无向边。

 [`Graphs`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Graphs.html) 类提供了一些基本的工具方法，如复制和比较图等。

## 图的类型

`common.graph` 提供了三个顶层的图接口，根据边的表示不同，分为 `Graph`、`ValueGraph` 和 `Network`，它们都是兄弟或叔侄，即它们中任何两个都没有继承关系。

<div align="center"><img src="./assets/Snipaste_2020-01-07_15-18-40.png"></img></div>

从上图可见，这些顶层接口都扩展了 [`SuccessorsFunction`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/SuccessorsFunction.html) 和 [`PredecessorsFunction`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/PredecessorsFunction.html) 接口。这两个接口旨在用作图论算法的参数类型（例如广度优先遍历），这些算法仅需要一种访问图中一个节点的后继/前继节点的方式。

当图的所有者已经具有适当的图的表示方式，并且不特别希望只为了运行一种图算法将这种表示方式序列化为 `common.graph` 时，这两个接口就能发挥作用了。

<div align="center"><img src="./assets/Snipaste_2020-01-07_15-27-45.png"></img></div>

### Graph

接口签名：

```java
/**
 * @param <N> Node parameter type
 */
public interface Graph<N> extends BaseGraph<N> {}
```

[`Graph`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Graph.html) 是最简单、最基础的图类型。它定义了基本的用于处理节点与节点间关系的操作，如 `successors(node)`、`adjacentNodes(node)` 和 `inDegree(node)`。Graph 中的节点是唯一的，实际上，Graph 用 Set 保存其所有节点。

`Graph` 的边是完全匿名的；它们仅根据其端点进行定义。

对于使用示例 `Graph<Airport>`：其边连接了机场，机场是唯一的，机场之间可以直飞。

### ValueGraph

接口签名：

```java
/**
 * @param <N> Node parameter type
 * @param <V> Value parameter type
 */
@Beta
public interface ValueGraph<N, V> extends BaseGraph<N> {}
```

可以看到，[`ValueGraph`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/ValueGraph.html) 的类签名上使用了两个泛型标记，这是因为其中每条边都关联了用户的指定值。这些值不必是唯一的（就像节点一样）。`ValueGraph` 和 `Graph` 之间的关系是类似于 `Map` 和 `Set`；`Graph`的边是一组端点对（EndpointPair），`ValueGraph`的边是从对端点到关联值的映射。

`ValueGraph` 具有 [`Graph`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Graph.html) 中所有与节点有关的方法，但是添加了一些方法来检索指定边的关联值。

`ValueGraph` 提供一个叫 `asGraph()` 的方法用来返回的 `ValueGraph` 的 `Graph` 视图 。这允许对 `Graph` 实例进行操作的方法也可以对 `ValueGraph` 实例起作用。

用例示例：`ValueGraph<Airport, Integer>`，其边值表示 `Airport` 在边连接的两个机场间移动所需的时间。

### Network

接口签名：

```java
/**
 * @param <N> Node parameter type
 * @param <E> Edge parameter type
 */
public interface Network<N, E> extends SuccessorsFunction<N>, PredecessorsFunction<N> {}
```

[`Network`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Network.html) 拥有 `Graph` 中所有的节点相关的方法，但增加了用于描述边及节点与边之间关系的方法，如 `outEdges(node)`、`incidentNodes(edge)` 和 `edgesConnecting(nodeU, nodeV)`。

像所有图类型中的节点一样，`Network`的边是唯一的。边的唯一性约束允许 [`Network`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Network.html) 本地支持平行边以及用于描述边与边间及节点与边间的关系的方法。

[`Network`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Network.html) 提供一个叫 `asGraph()` 的方法用于返回的 `Network` 的 `Graph` 视图，这允许对 `Graph` 实例进行操作的方法也可以对 `Network` 实例起作用。

用例示例：`Network<Airport Flight>`，其中边表示从一个机场到另一个机场可以乘坐的特定航班。

### 选择正确的图类型

三种图形类型之间的本质区别在于边的表示形式。

- Graph：边是节点间的匿名连接，没有其自身的标识或属性。你应该使用`Graph`，如果每一对节点由最多一个边连接，而你不需将任何信息与边相关联。

- ValueGraph：每条边都有相关值（例如，边缘权重或标签），这些值可能是唯一的也可能不是唯一的。如果图中每对节点最多由一条边连接，并且需要将某些信息（例如，边权重）与边关联，则应该使用 `ValueGraph`。

- Network 每条边都有唯一的关联值。如果边对象是唯一的（请注意，此唯一性允许 `Network` 支持平行边），并且希望能够将边与值进行关联。则应该使用 `Network`。

## 构建图实例

`common.graph` 中使用了 Builder 设计模式，其提供的实现类不是公共的。这减少了用户需要了解的公共类型的数量，并使得导航内置实现所提供的各种功能变得更加容易，而不会让那些只想创建图形的用户迷失在众多的实现中。

为了创建内置的图类型的实现，需要使用相应的 Builder 类实例：GraphBuilder，ValueGraphBuilder，或NetworkBuilder。例子：

```java
// 创建可变图
MutableGraph<Integer> graph = GraphBuilder.undirected().build();
MutableValueGraph<City, Distance> roads = ValueGraphBuilder.directed().build();

MutableNetwork<Webpage, Link> webSnapshot = NetworkBuilder.directed()
    .allowsParallelEdges(true)
    .nodeOrder(ElementOrder.natural())
    .expectedNodeCount(100000)
    .expectedEdgeCount(1000000)
    .build();

// Creating an immutable graph
ImmutableGraph<Country> countryAdjacencyGraph =
    GraphBuilder.undirected()
        .<Country>immutable()
        .putEdge(FRANCE, GERMANY)
        .putEdge(FRANCE, BELGIUM)
        .putEdge(GERMANY, BELGIUM)
        .addNode(ICELAND)
        .build();
```

- 你可以通过以下两种方式之一获得图 Builder 实例：
 
   - 调用静态方法 `directed()` 或`undirected()`。`Builder` 提供的每个 Graph 实例将是有向的或无向的。
   - 调用静态方法 `from()`，它会为你提供了一个基于现有图实例的 `Builder` 实例。
 
 - 创建`Builder`实例后，可以选择指定其他特征和功能。
 
 - 建立可变图
 
   - 你可以在同一 `Builder` 实例上多次调用 `build()` 方法，以使用相同的配置构建多个图形实例。
   - 你无需在 `Builder` 实例上指定元素类型。在图类型上指定即可。
   - 该 `build()` 方法返回 `Mutable` 相关图类型的子类，该子类提供了变异方法，有关更多信息，请参见 [“ `Mutable`和`Immutable`图形”](https://github.com/google/guava/wiki/GraphsExplained#mutable-and-immutable-graphs)。
 
 - 建立不可变图
 
   - 你可以在同一`Builder`实例上多次调用 `immmutable()` 方法，利用相同配置，创建多个 `ImmutableGraph.Builder` 对象。
   - 你需要在 `immutable` 调用中指定元素类型。

