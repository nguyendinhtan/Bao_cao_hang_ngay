# CookBook
- Tập trung nhiều hơn:  cookbook tập trung vào một khía cạnh cụ thể của Vue,  thay vì phải đưa ra một cái nhìn tuổi quan chung như trong hướng dẫn.
- Đi sâu bên trong: cookbook bao gồm các ví dụ phức tạp hơn , kết hợp các tính năng theo những cách thú vị, công thức có thể dài và chi tiết, cụ thể hơn. 
- JavaScript: Các tính năng thiết yếu (bao gồm ES6 / 2015+) có thể được khám phá và giải thích , giúp xây dựng vue tốt hơn.
- Khám phá hệ sinh thái: Khám phá chi tiết bên trong các thư viện hệ sinh thái 

 ## <span style="color: green; font-weight: bold"> Adding Instance Properties</span>
- Prototype là một đối tượng đặc biệt mặc định là thuộc tính của các function trong Javascript. Prototype chứa bản copy các thuộc tính của function, nó được gán lại cho các object được khởi tạo từ function đó qua thuộc tính __proto__. Prototype giúp chúng ta thực hiện kế thừa trong Javascript đồng thời để thêm động các thuộc tính chia sẻ cho tất cả các object.
- Có thể dữ liệu mà bạn muốn sử dụng trong nhiều thành phần, nhưng bạn ko muốn thay đổi phạm vi toàn cầu. Trong những trường hợp này bạn có thể cung cấp cho chúng từng phiên bản Vue bằng cách xác định chúng trên nguyên mẫu




> Ví dụ : khi ta ko dùng prototype

```
Function Student () {
	This.name = “jone”;
	This.gender = “male”;
}
Var studObj1 = new Student();
studObj1.age = 15;
alert(studObj1.age); //15
var studObj2 = new Student();
alert(studObj2.age); // undefined
```
> Khi dùng prototype: 
```
Function Student () {
	This.name = “jone”;
	This.gender = “male”;
}
Student.prototype.age = 15;
Var studObj1 = new Student();
studObj1.age = 15;
alert(studObj1.age); //15
var studObj2 = new Student();
alert(studObj2.age); // 15
```
ví dụ trên prototype trỏ đến thẳng đến đối tượng, function . tất cả các thể hiện đều có chung . chính vì vậy tương đương việc ta khai báo biến age ở trong Function Student.
> vi du 2:

```Vue.prototype.$appName = 'My App'```

  Bây giờ `$appName` có sẵn trên tất cả các phiên bản Vue , ngay cả trước khi tạo , nếu chúng ta chạy :
  ```
  new Vue({
  beforeCreate: function() {
    console.log(this.$appName)
  }
})
```
* Sau đó “My app” sẽ hiển thị trong Console!
  * 
  -	Tầm quan trọng của các thuộc tính phạm vi:

    $appName : $ là một quy ước Vue sử dụng cho các thuộc tính có sẵn cho tất cả các trường hợp. Điều này tránh xung đột với bất kỳ dữ liệu, thuộc tính hoặc phương pháp được xác định nào.
    thậm chí có thể sử dụng quy ước của riêng mình nếu bạn muốn, chẳng hạn như $_appNamehoặc ΩappName, để ngăn chặn ngay cả xung đột với các plugin hoặc các tính năng trong tương lai.
  -	The Context of Prototype Methods: 

    các phương thức được thêm vào một prototype trong javascript có được bối cảnh của thể hiện . có nghĩa là ta có thể sử dụng This để truy cập dữ liệu , thuộc tính được tính toán , phương thức hoặc bất kì thứ gì khác được xác định trên instace
> Ví dụ: phương thức reverseText
```
Vue.prototype.$reverseText = function(propertyName) {
  this[propertyName] = this[propertyName]
    .split('')
    .reverse()
    .join('')
}

new Vue({
  data: {
    message: 'Hello'
  },
  created: function() {
    console.log(this.message) // => "Hello"
    this.$reverseText('message')
    console.log(this.message) // => "olleH"
  }
})
```
`Lưu ý rằng context binding sẽ không hoạt động nếu sử dụng “=>” trên ES6, vì chúng hoàn toàn liên kết với phạm vi cha mẹ của chúng `
```
Vue.prototype.$reverseText = propertyName => {
  this[propertyName] = this[propertyName]
    .split('')
    .reverse()
    .join('')
}
```
Sẽ sảy ra lỗi :  
`Uncaught TypeError: Cannot read property 'split' of undefined`
- ### When To Avoid This Pattern:
  - Sử dụng mẫu này khá an toàn , gần như ko có khả năng sảy ra lỗi. Tuy nhiên đôi khi có thể gây nhầm lẫn với các nhà phát triển khác , họ có thể thấy this.$http, ví dụ họ ko biết về tính năng Vue này , họ chuyển sang một dự án khác và bối rối khi $http không được xác định hoặc có thể họ muốn Google làm một cái gì đó nhưng không thể tìm thấy kết quả bởi vì họ không nhận ra họ thực sự sử dụng Axios dưới một bí danh. 
-	### Alternative Patterns (mô hình thay thế):
    - Trong các ứng dụng không sử dụng hệ thống module(thông qua webpack hoặc browserify), nếu những gì bạn muốn thêm không liên quan đến vue thì đây có thể là một lựa chọn để tiếp cận 
  >vi du: 
  ```
  var App = Object.freeze({
  name: 'My App',
  version: '2.1.4',
  helpers: {
    // This is a purely functional version of
    // the $reverseText method we saw earlier
    reverseText: function(text) {
      return text
        .split('')
        .reverse()
        .join('')
    }
  }
})
```
Bây giờ nguồn gốc của các thuộc tính được chia sẻ rõ ràng hơn , ứng dụng có thể được sử dụng ở bất kì đâu trong mã của bạn, cho dù nó liên quan đến vue hay không. Điều đó bao gồm việc đính kèm các giá trị trực tiếp vào các tùy chọn thể hiện thay vì phải nhập một hàm để truy cập các thuộc tính trên này: 
```new Vue({
  data: {
    appVersion: App.version
  },
  methods: {
    reverseText: App.helpers.reverseText
  }
})
```
- ### Khi sử dụng module system:
  <p>Khi bạn có quyền truy cập vào hệ thống module, bạn có thể dễ dàng sắp xếp mã được chia sẻ thành các module, sau đó yêu cầu require/ import các module đó bất cứ nơi nào họ cần. mặc dù chắc chắn sẽ dài dòng hơn, cách tiếp cận này chắc chắn là dễ bảo trì nhất đặc biệt là khi làm việc với các nhà phát triển khác hoặc xây dựng một ứng dụng lớn. </p>

## <span style="color: green; font-weight: bold">Form Validation</span>

*  Xác thực form là việc buộc người dùng khi nhập thông tin trên form cần tuân theo quy tắc mà nhà nhát triển đặt ra.
  
*  Form validation được hỗ trợ bởi trình duyệt, nhưng đôi khi các trình duyệt khác nhau sẽ xử lý mọi thứ theo cách khiến việc dựa vào nó hơi khó khăn. Ngay cả khi form validation được hỗ trợ hoàn hảo , có thể đôi khi cần validate tùy chỉnh và giải pháp dựa trên vue thủ công hơn có thể sẽ phù hợp hơn 
> Ví dụ:  Using Custom Validation:

```
<form
  id="app"
  @submit="checkForm"
  action="https://vuejs.org/"
  method="post"
  novalidate="true"
>

  <p v-if="errors.length">
    <b>Please correct the following error(s):</b>
    <ul>
      <li v-for="error in errors">{{ error }}</li>
    </ul>
  </p>

  <p>
    <label for="name">Name</label>
    <input
      id="name"
      v-model="name"
      type="text"
      name="name"
    >
  </p>

  <p>
    <label for="email">Email</label>
    <input
      id="email"
      v-model="email"
      type="email"
      name="email"
    >
  </p>

  <p>
    <label for="movie">Favorite Movie</label>
    <select
      id="movie"
      v-model="movie"
      name="movie"
    >
      <option>Star Wars</option>
      <option>Vanilla Sky</option>
      <option>Atomic Blonde</option>
    </select>
  </p>

  <p>
    <input
      type="submit"
      value="Submit"
    >
  </p>

</form>
```

`Js:`

```
const app = new Vue({
  el: '#app',
  data: {
    errors: [],
    name: null,
    email: null,
    movie: null
  },
  methods: {
    checkForm: function (e) {
      this.errors = [];

      if (!this.name) {
        this.errors.push("Name required.");
      }
      if (!this.email) {
        this.errors.push('Email required.');
      } else if (!this.validEmail(this.email)) {
        this.errors.push('Valid email required.');
      }

      if (!this.errors.length) {
        return true;
      }

      e.preventDefault();
    },
    validEmail: function (email) {
      var re = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
      return re.test(email);
    }
  }
})
```
- Mô hình thay thế: 
  <p style="color: red">có một số thư viện Vue tuyệt vời sẽ xử lý rất nhiều việc này cho bạn.  Chuyển sang thư viện đóng gói sẵn có thể ảnh hưởng đến kích thước cuối cùng của ứng dụng của bạn, nhưng lợi ích có thể rất lớn.  Bạn có mã (rất có thể) được kiểm tra nhiều và cũng được cập nhật thường xuyên.</p>

  * [vuelidate](https://github.com/monterail/vuelidate)
  * [VeeValidate](https://logaretm.github.io/vee-validate/)

- VeeValidate: 

      thư viện VeeValidate.
```
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>Vue Template Form Validation</title>
</head>
<body>
</body>
<!-- include the Vue.js library -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.5.9/vue.js"></script>
<!-- include the VeeValidate library -->
<script src="https://cdn.jsdelivr.net/npm/vee-validate@latest/dist/vee-validate.js"></script>
<script>
// Notify vue about the VeeValidate plugin
 Vue.use(VeeValidate);
</script>
```
#### VeeValidate Rules: 
* Quy tắc VeeValidate là quy tắc đặt giới hạn hoặc điều kiện cho những gì có thể được nhập vào một hoặc nhiều trường.
Quy tắc xác thực được kiểm tra khi bạn cập nhật một bản ghi có chứa các trường yêu cầu xác thực.
Nếu quy tắc bị vi phạm, một lỗi có thể xảy ra. Để sử dụng quy tắc bạn chỉ cần áp dụng chỉ thị v-validate trên đầu vào của bạn và truyền một giá trị chuỗi là danh sách các xác nhận được phân tách bằng một dấu "|". 
>Ví dụ:sử dụng các quy tắc required &email:

`<input v-validate="'required|email'" type="text" name="email">`

>Ngoài ra bạn còn có thể validate 1 cách link hoạt hoạt hơn

`<input v-validate="{ required: true, email: true, regex: /[0-9]+/ }" type="text" name="email">`

Bây giờ mỗi khi đầu vào thay đổi, trinh xác nhận sẽ chạy danh sách xác nhận từ trái qua phải. Để xác thực lỗi dữ liệu đầu vào.
#### VeeValidate Errors: 
* Theo mặc định VeeValidate cung cấp cho chúng ta một biến lỗi được đưa vào phần dữ liệu của thành phần Vue bởi plugin. Khi xác thực mẫu không thành công, VeeValidate biến lỗi này với một mảng chứa các đối tượng xác thực không thành công,
có thể được truy cập theo cách sau:

    ```
    //check if an input has errors
    this.errors.has(Inputname)
    //return the first error of an input
    this.errors.first(Inputname)
    ```
`Tuy nhiên, để sử dụng phương pháp này, thuộc tính tên phải được chỉ định.
Bạn có thể sử dụng data-vv-name cho một số lý do, bạn có thể sử dụng thuộc tính name trong mẫu của mình.
`









