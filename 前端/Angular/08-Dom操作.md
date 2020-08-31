> 08 - Angular 中的 Dom 操作以及@ViewChild、Angular 执行 css3 动画  



## 一、Angular 中的 dom 操作（原生 js）   

```typescript
ngAfterViewInit(){
	var boxDom:any=document.getElementById('box');
	boxDom.style.color='red';
}
```



## 二、Angular 中的 dom 操作（ViewChild）  

```typescript
import { Component ,ViewChild,ElementRef} from '@angular/core';
@ViewChild('myattr') myattr: ElementRef;
```

```html
<div #myattr></div>
```

```typescript
ngAfterViewInit(){
	let attrEl = this.myattr.nativeElement;
}
```



## 三、父组件通过 ViewChild 调用子组件的方法

#### 3.1 调用子组件给子组件定义一个名称

```html
<app-footer #footerChild></app-footer>
```

#### 3.2 引入 ViewChild

```typescript
import { Component, OnInit ,ViewChild} from '@angular/core';
```

#### 3.3 ViewChild 和刚才的子组件关联起来

```typescript
@ViewChild('footerChild') footer;
```

#### 3.4 调用子组件  

```typescript
run(){
	this.footer.footerRun();
}
```

