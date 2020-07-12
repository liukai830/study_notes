> 05 - Angular中Dom操作 以及表单（ input、checkbox、radio、select、 textarea ）结合双休数据绑定实现在线预约功能

双向绑定首先要在form下才能使用

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

// 双向绑定
import { FormsModule } from '@angular/forms'

import { AppComponent } from './app.component';
import { FormComponent } from './components/form/form.component';

@NgModule({
  declarations: [
    AppComponent,
    NewsComponent,
    HomeComponent,
    HeaderComponent,
    FormComponent
  ],
  imports: [
    BrowserModule,
    FormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```



form.components.ts

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-form',
  templateUrl: './form.component.html',
  styleUrls: ['./form.component.scss']
})
export class FormComponent implements OnInit {

  constructor() { }

  ngOnInit(): void {
  }

  public peopleInfo:any={
    username: 'admin',
    sex: '2',
    cityList: ['北京','上海','深圳'],
    city: '上海',
    hobby:[{
      title:'吃饭',
      checked:false
    },{
      title:'睡觉',
      checked:false
    },{
      title:'敲代码',
      checked:true
    }],
    mark: '备注'
  }

  doSubmit(){
    console.log(this.peopleInfo);
  }
}
```

form.components.html

```html
<h2>人员登记系统</h2>
<div class="people_list">
  <ul>
    <li>姓 名：<input type="text" id="username" [(ngModel)]="peopleInfo.username" /></li>
    <li>性 别：
      <input type="radio" value="1" name="sex" id="sex1" [(ngModel)]="peopleInfo.sex"> <label for="sex1">男 </label>　　　
      <input type="radio" value="2" name="sex" id="sex2" [(ngModel)]="peopleInfo.sex"> <label for="sex2">女 </label>
    </li>
   <li>
    城 市：
      <select name="city" id="city" [(ngModel)]="peopleInfo.city">
          <option [value]="item" *ngFor="let item of peopleInfo.cityList">{{item}}</option>
      </select>
    </li>
    <li>
        爱 好：
        <span *ngFor="let item of peopleInfo.hobby;let key=index;">
            <input type="checkbox"  [id]="'check'+key" [(ngModel)]="item.checked"/> <label [for]="'check'+key"> {{item.title}}</label>
            &nbsp;&nbsp;
        </span>
     </li>
     <li>
       备 注：
       <textarea name="mark" id="mark" cols="30" rows="10" [(ngModel)]="peopleInfo.mark"></textarea>
     </li>
  </ul>
  <button (click)="doSubmit()" class="submit">获取表单的内容</button>

  <pre>
    {{peopleInfo | json}}
  </pre>

</div>
```

