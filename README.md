# observeObject
项目总结（自我赶脚，解释的蛮清晰蛤蛤蛤）


1：forEach
	array.forEach(function (method, index, array) {
                console.log(this);
            }, this);

不传this的话，在forEach中this永远为undefined


2：关于代码思路，注释中已经写得很清楚，这里再总结一下
	（1）：要监控一个对象，意思就是，在改变这个属性的时候，可以调用我们的cb，那么意思就是我们要知道属性改变的发生点，并提前部署好，这样当属性改变时才会执行我们提前部署好的语句。
	那么属性改变时，会发生什么呢，当然是触发setter函数，那么这里就是属性改变从而引起的触发点，我们在setter函数里部署好我们的语句，比如调用callback，这样的话，当属性一改变，setter执行，从而执行我们的代码。

	（2）：当监控的属性为对象或数组时，递归对该属性对象继续监控（就跟监控初始对象一样）
	（3）：当我们想要在对属性进行push.pop.shift等操作时，也能监控到，那么毫无疑问，就要对这些方法进行修改，那么这些方法在Array的原型上，我们当然不能动啊，影响到其它数组怎么办。
	所以一般遇到以下这种情况：当我们想改变某种操作的行为，可以将该操作和我们想做的操作合并起来作为一个函数，就像一个函数中嵌着另一个函数，这不就是，在做里面的操作同时，又做我们的操作吗。
	所以思路就是，写一个函数：
proxyPush() {
	Array.push.call(null, '');
	//do something else
}

	那么我们已经准备好该函数了，接下来的问题就是，怎样让数组调用push时，调用的是这个我们写的push，而不是Array.prototype.push。

解决思路如下：
在执行arr.push(1)时，是顺着该对象的__proto__找到Array.prototype，执行这里的push，那么我们想要让它执行新写的push，就把__proto__指向改到一个新对象上，把新写的push函数，加在这个对象上，然后这个对象的__proto__再指向Array.prototype，其实就是半路截个胡，中间加个代理，这样自然就会顺着原型链执行到新的push，在这个新的push中，又正确的执行了原本的push方法，又执行了我们的代码（如调用callback），完美


	（4）：得到每个属性的路径。比如对于下面的例子
var data = {
        name: 'night',
        age: '21',
        hobby: {
            id: 1,
            name: 'dance',
            level: 'high',
            lucky_number: [1, 2, 7, 4]
        },
        lucky_number: [1, 2, 7, 4]
    }

name属性的路径就为[data,hobby,name]

观察这个数据结构，就像树一样，每个属性，就是一个节点，那么可以转换为找树中节点的位置，OK，解决了，那就是树的深层遍历算法嘛~

	
