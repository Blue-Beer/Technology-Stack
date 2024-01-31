# Code Smell

1. **重复代码**
    对代码的简单复制粘贴，开发时复制粘贴很方便，但是一旦此类代码出现问题需要修改，很容易在修改时遗漏先前某个复制过的代码。
    **解决**：

    - 单一类中的重复代码可以提炼出一个方法 Extract Method。
    - 兄弟类中重复出现的代码可以提炼出这个方法并将这一方法放在这些兄弟类都要继承的父类中 Extract Method。
    - 关联性不大的类中出现的重复代码可以使用一个新的类来提炼这个重复代码，这些类统一依赖这个类 Extract Method。

2. **长函数**
    横向的函数名 过长导致方法的表意难以理解。
    纵向的单一函数内的代码量过大，修改容易顾此失彼。
    **解决**：

    - 横向长函数：在团队内统一使用一种代码格式化的标准，使用该标准修正代码的横向长度。
    - 纵向长函数：
      - 减少不必要的临时变量的使用，改为使用赋予临时变量值的表达式 Inline Temp 。
      - 长函数往往指向了该函数的职责不够单一，过程性地描述了一个业务的多个环节，所以需要提炼新的方法来单一化函数职责 Extract Method。
  
3. **过大的类** **发散式变化**
    过大的类往往指向了其职责不单一，并且在问题定位时很困难。因为其职责不单一，会造成多种其依赖的类的变动都会引起这个类的变动。

    **解决**：

    - 对类进行拆分：
      - 对这个类中可以表达一定体量业务的属性及其方法挪出一个新类  Extract Class。
      - 这个类中如果有一些方法普遍不需要类中的一些属性，那这些方法就应该被提炼成一个相关的子类 Extract Subclass。

4. **过长的参数列表**
    可读性差，后续修改困难

    **解决**：

    - 将多个参数统合为一个对象进行传递。

5. **Switch 语句**
    switch语句实现的业务逻辑无法满足开闭原则，每一次新增业务都需要修改原本的代码。

    **解决**：
    - 对于符合多态的switch，使用多态代替switch语句。

6. **未来性代码**
    敏捷开发要做的是对现有需求的快速迭代，而不是为了考虑未来的可能出现的需求影响当前的开发结构。

    **解决**：
    - 不要假设业务方将来会产生的需求。

7. **多余的注释**
    喃喃自语，误导性注释，日志式注释，废话注释，用注释来解释变量意思，用来标记位置的注释，类的归属的注释，注释掉的代码

    **解决**：
    - 用合适的类，方法，属性的命名代替注释的功能

8. **霰弹式修改**
    一个类的修改导致其他的多个类也要发生修改。一对多的修改链指向的是一种强耦合。

    **解决**：
    - 将代码有联系的方法或属性移动到这一个类中 Move Field / Move Method
    - 创建新的类来中介修改的变动 Extract Class

9. **拒绝的馈赠**
    当子类集成基类的时候，基类的一些方法子类不需要但是被迫继承。实现了不需要的方法，对后续的开发造成了误导。

    **解决**：
    - 剔除子类不需要的方法。