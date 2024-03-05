
```json

  "date": "2023.12.24 22:10",
  "tags": ["Framework","DDD"]

```

## Domain-driven design(DDD)

- 软件核心复杂性应对之道
- 业务如何运行,软件就如何构建, 所有应用的归宿都是变成DDD的结构
- DDD将应用看成是领域之间的组合,让架构调整贯穿整个项目周期
- 领域划分的核心就是边界划分

## 领域驱动设计（DDD）四层架构

### 1. 用户界面层（UI层）
- 负责与用户进行交互，接收用户输入，并向用户显示信息。
- 包括用户界面元素、页面和其他用户交互的组件。
- 在DDD中，UI层主要负责将用户的请求传递给应用服务层。

## 2. 应用服务层（Application层）
- 扮演协调领域层和基础设施层之间交互的角色。
- 包含应用服务，负责协调领域对象的操作，确保按业务规则进行正确交互。
- 负责事务管理、安全性和其他与应用的跨领域关注点相关的任务。

## 3. 领域层（Domain层）
- 整个系统的核心，包含领域对象、实体、值对象、聚合根等。
- 设计直接映射业务领域的概念和规则。
- 负责执行业务逻辑，确保系统的行为符合业务需求。
- DDD 提倡使用领域模型来表示和解决业务问题。

## 4. 基础设施层（Infrastructure层）
- 提供支持整个应用程序的技术基础结构，包括数据库、消息队列、外部服务等。
- 任务是将领域层和其他层连接在一起，使它们协同工作。
- 处理一些与具体技术相关的事务，例如数据访问、日志记录等。

这种四层架构有助于将不同层次的关注点分离开来，使系统更易于理解、扩展和维护。每一层都有其特定的职责，从而降低了系统的复杂性，同时也促进了更好的模块化和可测试性。在实施DDD时，开发团队应该根据具体的业务需求和系统特点进行适当的调整。

# 防腐层设计建议

在领域驱动设计（DDD）中，"防腐层"是一个重要的概念，用于在不同的子域（Bounded Context）之间进行数据交互时采取一些策略和模式，以防止数据模型的混淆和依赖关系的混乱。以下是一些建议，可用于设计防腐层：

## 1. 明确定义子域边界
- 在设计防腐层之前，首先需要明确定义子域的边界。了解不同子域的边界有助于明确防腐层的作用范围。

## 2. 使用适配器模式
- 适配器模式是防腐层设计中常用的模式之一。通过适配器，可以将一个子域的模型适应到另一个子域的模型上，从而在两个子域之间建立一种转换层，有助于解耦两个子域。

## 3. 定义映射规则
- 明确不同子域之间的映射规则，包括字段的映射、值对象的映射、枚举的映射等。有清晰的映射规则可以减少数据转换的复杂性。

## 4. 使用 Anti-Corruption Layer（ACL）
- Anti-Corruption Layer 是防腐层的一种实现方式，负责处理不同子域之间的交互。在ACL中，可以进行数据的验证、转换和映射，以确保数据在两个子域之间的一致性。

## 5. 避免直接依赖
- 防腐层的目标是防止不同子域之间的直接依赖关系。避免在一个子域中直接调用另一个子域的服务或访问其数据库，通过防腐层建立松耦合的关系。

## 6. 定期同步和协调
- 定期审查和更新防腐层，以确保其仍然符合业务需求。业务变化可能导致子域之间的数据交互方式变化，因此防腐层需要根据需要进行调整。

## 7. 文档化和测试
- 为防腐层编写文档，明确其设计原则和使用规范。编写测试来验证防腐层的正确性，确保其能够按照预期工作。

## 8. 考虑异步通信
- 在不同子域之间使用异步通信机制，如消息队列，有助于降低实时性的要求，减轻子域之间的直接依赖关系，同时提高系统的可伸缩性。

设计防腐层是一个依赖具体业务情境
```cpp

// Order.hpp
#include <vector>

class Order {
public:
    Order(int orderId) : orderId(orderId) {}

    void addProduct(int productId, int quantity);

private:
    int orderId;
    std::vector<std::pair<int, int>> products;  // 商品ID和数量的列表
};

// InventoryItem.hpp
class InventoryItem {
public:
    InventoryItem(int productId, int availableQuantity) : productId(productId), availableQuantity(availableQuantity) {}

    bool reserve(int quantity);

private:
    int productId;
    int availableQuantity;
};

为了防止订单管理子域直接依赖库存管理子域，我们引入防腐层。我们创建一个名为 InventoryService 的类，该类负责与库存管理子域进行交互，通过适配器模式来进行防腐：

// InventoryService.hpp
#include "InventoryItem.hpp"

class InventoryService {
public:
    bool reserveInventory(int productId, int quantity);

private:
    InventoryItem getInventoryItem(int productId);  // 通过库存管理子域获取库存信息的适配器方法
};

// InventoryService.cpp
#include "InventoryService.hpp"

bool InventoryService::reserveInventory(int productId, int quantity) {
    InventoryItem inventoryItem = getInventoryItem(productId);
    return inventoryItem.reserve(quantity);
}

InventoryItem InventoryService::getInventoryItem(int productId) {
    // 此处省略通过库存管理子域获取库存信息的逻辑
    // 这可能包括调用库存管理子域的服务、API等
    return InventoryItem(productId, /* 获取的库存数量 */);
}

现在，订单管理子域只需要与 InventoryService 交互，而不需要直接依赖库存管理子域。这种设计有助于防止子域之间的直接依赖关系，提高系统的灵活性和可维护性.

```

# 抽象中间件 

-  防腐层 隔离第三方组件
-  摆脱技术框架限制,提供无限可能
-  比如就是 传统的Service直接调用Dao,但是现实这个Dao层不能和Service直接发生关系
-  隔离所谓的变化, 跨多个业务的动作,交给每个实体去完成.抽象的接口里的方法一定要传入实体
-  service只调用动作实现业务场景抽象,动作归根到实体里面.

```
public interface AccountTransferService{
  void transfer(Account sourceAccount,Account targetAccount,Money money);
}

public class AccountTransferServiceImpl implements AccountTransferService{
  public void transfer(Account sourceAccount,Account targetAccount,Money money){
    sourceAccout.deposit(money);
    targetAccout.withdraw(money);
  }
}

```


# 第四步: 用领域服务封装多实体逻辑 - 使用领域服务,封装跨实体业务,保证实体纯粹性.

- 只负责组装场景, 不负责功能实现.

￼![图片](assets/images/IMG_1.png)

￼ ![图片](assets/images/IMG_2.png)

- 聚合的作用: 聚合是确保这些领域对象在实现共同的业务逻辑时,能保证数据的一致性.
- 聚合根的实现 可以减少依赖关系, 访问该聚合根代表访问其方法.
