### 1.v-if和v-for哪个优先级高？如果两个同时出现，应该怎么优化得到更好的性能？

1. 当它们处于同一节点时，v-for比v-if优先级更高。因为v-for需要将模板转换为VNode，而v-if是VNode中负责控制是否渲染成真实Dom的对象，所以v-for比v-if优先级更高
2. 优化方案：
   - 如果是为了过滤掉数据结构中某个状态数据的话，可以使用computed计算属性来对v-for要迭代的数据进行过滤，则可以省略v-if判断，只迭代出想要的数据项。
   - 如果v-if是为了控制v-for渲染后的整体结果显示隐藏的话，那么建议将v-if提升至v-for的父容器上，这样可以避免v-if条件不成立时，仍然会去进行v-for的渲染，造成无意义的性能浪费。

