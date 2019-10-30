# 利用脚手架编写的一个拼图小游戏

 ## 思路：
 1. 先用`v-for`循环遍历数组`correctPoints`生成n（第一关为9个，以9个为例）个长宽相等小方块，记得加上`key`
    值为`item.id`
 2. 设置背景图为父组件传过来的`img`路径，则每个小方块对应背景图的一块，此时整个图完整，作为正确位置保存
 3. 定义乱序后的数组`blockPoints`,基于`correctPoints`和数组的`sort`方法生成，
    但最后一个方块的位置不变，且其`opacity=0`，作为空白方块，用以交换位置
 4. 循环遍历数组`blockPoints`，生成乱序后的拼图
    + 其实小方块对应的背景图不变，改变的是其`left`和`top`值
 5. 空白方块与有图方块交换位置的条件：有一条边与空白方块相邻的有图方块，才能与其交换位置
 6. 检查是否通关：比较当前数组里有图小方块的位置与`correctPoints`里的位置，用数组的`every`方法
    只有全部方块的`left`值与`top`值都跟`correctPoints`里的相等时，才能通关
 7. 游戏通关后，最后一个小方块（即空白方块）的背景图显示，并确定是否继续下一关
 8. 如果选择继续下一关，则通过自定义事件告知父组件，更新拼图数据对象后，
    通过属性传值给子组件，渲染新一关的页面

## 如何得到`correctPoints`?
1. 根据父组件传过来的拼图长度与宽度，还有拼图行数、列数，可计算得到小方块的宽高 `blockWidth`
2. 经过分析，小方块的`left`值都是`blockWidth`的倍数,`top`值则是`blockHeight`的倍数
3. 故通过两层for循环，得到每个小方块的`left`和`top`值，同时也用`date`对象生成了唯一的id值，
   作为一个个对象添加到数组`correctPoints`中
 `{   x:j*blockWidth,`
    ` y:i*blockHeight,`
   `  id:new Date().getTime()+Math.random()*100}`

## 如何得到`blockPoints`?
1. 先定义`correctPoints`数组
2. 取出最后一个小方块，作为空白方块 `const lastEle=correct[length-1];`
3. 定义一个新数组，存放用剩余运算符拷贝的`correctPoints`数组（不包括最后一个方块的数据对象）
   `const newArr=[...correct];`
4. 调用数组的`sort`方法，结合随机数，生成乱序数组 `newArr.sort(()=>Math.random()-0.5);`
5. 将最后一个小方块加到乱序数组后面，且透明度为0

## 方块位置交换
1. 点击方块时，触发`handleClick()`事件，通过`e.target`获取事件对象（即被点击的方块）
2. 用`ref`指定空白方块，`:ref="index===blockPoints.length-1? 'empty':'block'"`
   通过`this.$refs.empty[0]`得到空白方块对象 
3. 调用`checkBorder`函数，比较被点击方块和空白方块的`left` `top`值
4. 若符合条件，则直接交换两个方块的`left`和`top`值，完成位置的调换，实现拼图的移动
5. 交换完成后，调用`checkWin`函数，检查是否通关

## 调换位置的条件(画图可看出)
1. 只有一边与空白方块相邻的有图方块，才能与之调换位置
2. 若被点击方块与空白方块处于同一行，即`top`值相等时，如果`left`值相减的绝对值为`1*blockWidth`,则可交换
3. 若被点击方块与空白方块处于同一列，即`left`值相等时，如果`top`值相减的绝对值为`1*blockHeight`,则可交换

## 检查是否通关？
1. 通过 `:data-correctx="correctPoints[index].x"` 可拿到正确位置的`left`值，同理可拿到正确的`top`值
2. 通过 `const domArr=this.$refs.block;` 可拿到存放所有有图方块的一个数组，进而拿到它们的`left``top`值
3. 调用数组的`every`方法，对比有图数组的`left(top)`与正确数组的`left(top)`,只有当全部相等时，才算通关
4. 返回一个布尔值`flag`

## 是否继续下一关
1. 通关后调用`winGame`函数，且最后一个小方块的透明度为1
2. 调用`goNextLevel`函数
3. 通过`window.confirm`得到一个布尔值，用以确定是否继续下一关


## 继续下一关
1. 在`goNextLevel`函数中定义自定义事件 `this.$emit('nextLevel')`
2. 在父组件中定义`nextLevel`函数，改变索引值`index`
3. 改变后对`index`作出判断，不能高于拼图数据数组的长度
4. `index`修改后，传给子组件的拼图数据也改变，故拼图页面更新，进入下一关
5. 当完成最后一关时，询问是否重来一轮，若是，则令`index=0`，重新开局


