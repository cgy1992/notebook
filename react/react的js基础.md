# 使用箭头函数的好处

在箭头函数出现之前，每个新定义的函数都有它自己的 this 值。

例如在 react 中定义了一个计数器有以下代码：

```js
class Timer extends React.Component{

    constructor(props){
        super(props);

        this.handleClick = this.handleClick.bind(this);  //需要在这里绑定 将this 值传递过来

        this.state = {
            count: 0,
        };
    }

    handleClick(e){
        e.preventDefault();

        this.setState({
            count:this.state.count + 1,   // 绑定了以后这里才能使用 this.state
        });
    }

    render(){
        return(
          <div>
              <p>{this.state.count}</p>
              <a href="#" onClick={this.handleClick}>点我更新呀</a>
          </div>
        );
    }
}

export default Timer;
```

 如果上面的函数不进行绑定，则在 handleClick 内部的 this.setState 会报空，但是如果进行如下使用箭头函数，则没有问题：

```js
//构造函数中不进行绑定
// this.handleClick = this.handleClick.bind(this);

//修改 handleClick 方法

    handleClick(){

        this.setState({
            count:this.state.count + 1,
        });
    }
// 引入箭头函数
 <a href="#" onClick={()=>{this.handleClick()}}>点我更新呀</a>
```

这里，没有进行绑定，但是一样可以使用 react 的 state 等，因为：

> 箭头函数会捕获其所在上下文的  this 值，作为自己的 this 值