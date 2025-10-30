# 🧩 Định nghĩa kiểu dữ liệu cho CSS với `@property`

Từ lâu, chúng ta đã quen thuộc với **CSS custom properties** (biến CSS) — một công cụ mạnh mẽ để tái sử dụng giá trị trong style.  
Tuy nhiên, nhược điểm là các biến này **không có kiểu dữ liệu**, **thiếu khả năng kiểm tra hợp lệ**, và **không thể animation natively**.  

Giờ đây, với **`@property`**, mọi thứ đã thay đổi.  
`@property` cho phép định nghĩa **kiểu dữ liệu (type definitions)**, **giá trị mặc định**, và **cách kế thừa** cho custom properties — **hoàn toàn bằng CSS thuần**.

---

## ⚙️ Lợi ích chính của `@property`

`@property` mang lại **4 lợi ích lớn** so với custom properties thông thường:

1. ✅ **Type safety** – đảm bảo đúng kiểu dữ liệu  
2. 🔄 **Controlled inheritance** – kiểm soát khả năng kế thừa  
3. 🎞 **Native animation support** – hỗ trợ animation tự nhiên  
4. 🧠 **Integration with CSS Houdini & JS APIs** – tích hợp sâu với JavaScript và Houdini

---

## 1️⃣ Type Safety – Kiểm soát kiểu dữ liệu

Với `@property`, bạn có thể định nghĩa kiểu giá trị mà biến CSS chấp nhận.

```css
@property --toast-bg-color {
  syntax: "<color>";
  initial-value: white;
  inherits: false;
}
```

Khi đó, trình duyệt sẽ **kiểm tra hợp lệ giá trị**.  
Nếu giá trị không hợp lệ, nó sẽ **bị bỏ qua** và **quay về giá trị mặc định (`initial-value`)**.

Bạn thậm chí có thể định nghĩa kiểu dữ liệu cụ thể hơn:

```css
syntax: "white | green";
```

→ Chỉ cho phép đúng hai giá trị `"white"` hoặc `"green"`.  
Điều này giúp code CSS trở nên **an toàn và đáng tin cậy hơn nhiều**.

### 🔍 Ví dụ trực quan:

```html
<div class="toast">Hello World!</div>
```

```css
@property --toast-bg-color {
  syntax: "<color>";
  initial-value: white;
  inherits: false;
}

.toast {
  --toast-bg-color: lightgreen;
  background: var(--toast-bg-color);
  padding: 1rem;
  border-radius: 8px;
}

.toast:hover {
  --toast-bg-color: pink; /* Nếu sai kiểu, sẽ tự động bỏ qua */
}
```

> 🧠 Nếu bạn thử đặt giá trị `--toast-bg-color: 123px;`, trình duyệt sẽ **bỏ qua** vì không hợp lệ với kiểu `<color>`.

---

## 2️⃣ Controlled Inheritance – Kiểm soát kế thừa

Mặc định, **custom properties luôn kế thừa** từ phần tử cha, điều này có thể gây lỗi style không mong muốn khi làm việc với **nested components**.

Với `@property`, bạn có thể **quy định rõ ràng việc kế thừa**:

```css
inherits: false; /* hoặc true */
```

→ Quyết định xem giá trị có **lan xuống phần tử con** hay **giữ nguyên trong phạm vi hiện tại**.  
Tính năng này rất hữu ích khi bạn muốn **tạo token thiết kế (design tokens)** có phạm vi giới hạn, giúp component trở nên **độc lập và tái sử dụng dễ dàng hơn**.

### 🔍 Ví dụ minh họa:

```html
<div class="card">
  <div class="child">Child</div>
</div>
```

```css
@property --card-color {
  syntax: "<color>";
  inherits: false;
  initial-value: lightgray;
}

.card {
  --card-color: coral;
  background: var(--card-color);
  padding: 20px;
}

.child {
  background: var(--card-color); /* Không kế thừa từ cha */
}
```

> 💡 Trong ví dụ này, phần tử `.child` sẽ vẫn giữ màu mặc định `lightgray`, thay vì kế thừa `coral` từ `.card`.

---

## 3️⃣ Native Animation Support – Hỗ trợ animation tự nhiên

Trước đây, bạn không thể animation custom property vì trình duyệt **không biết cách nội suy giá trị** (interpolate).

Ví dụ:

```css
.toast__close {
  --rotate: 0deg;
  transform: rotate(var(--rotate));
  transition: --rotate 0.5s ease-in-out;
}

.toast__close:hover {
  --rotate: 180deg;
}
```

→ Kết quả: **Không có animation**!  
Trình duyệt không biết `--rotate` là kiểu gì nên không thể chuyển động mượt.

Nhưng với `@property`, bạn có thể khai báo rõ:

```css
@property --rotate {
  syntax: "<angle>";
  inherits: true;
  initial-value: 0deg;
}
```

Giờ đây, trình duyệt **hiểu rằng `--rotate` là một góc (`<angle>`)**,  
và có thể **tính toán quá trình chuyển đổi giữa hai giá trị** → animation hoạt động mượt mà. 🌀

### 🔍 Ví dụ minh họa đầy đủ:

```html
<button class="rotate-btn">⟳ Hover me!</button>
```

```css
@property --rotate {
  syntax: "<angle>";
  inherits: true;
  initial-value: 0deg;
}

.rotate-btn {
  --rotate: 0deg;
  transform: rotate(var(--rotate));
  transition: --rotate 0.5s ease-in-out;
}

.rotate-btn:hover {
  --rotate: 180deg;
}
```

> 🧩 Khi hover, nút sẽ xoay tròn một cách mượt mà – hoàn toàn bằng CSS thuần.

---

## 4️⃣ Integration with CSS Houdini & JS APIs – Tích hợp JavaScript

`@property` còn hỗ trợ **đăng ký biến có kiểu dữ liệu ngay trong JavaScript**, thông qua **CSS Houdini API**:

```js
window.CSS.registerProperty({
  name: "--rotate",
  syntax: "<angle>",
  inherits: true,
  initialValue: "0deg"
});
```

Điều này giúp bạn:
- Giao tiếp linh hoạt giữa CSS và JS  
- Xây dựng các hệ thống **style động**  
- Viết **animation hoặc hiệu ứng phức tạp** mà vẫn tối ưu hiệu năng

### 🔍 Ví dụ:

```html
<div class="ball"></div>
```

```css
.ball {
  width: 50px;
  height: 50px;
  background: dodgerblue;
  border-radius: 50%;
  transform: translateX(var(--x, 0px));
  transition: --x 0.5s ease-in-out;
}
```

```js
window.CSS.registerProperty({
  name: "--x",
  syntax: "<length>",
  inherits: false,
  initialValue: "0px"
});

document.querySelector(".ball").addEventListener("click", (e) => {
  e.target.style.setProperty("--x", "200px");
});
```

> 🧠 Khi click vào hình tròn, nó sẽ trượt sang phải 200px — tất cả được định nghĩa rõ ràng bằng kiểu `<length>`.

---

## ✨ Kết luận

`@property` giúp CSS:
- Trở nên **an toàn hơn về kiểu dữ liệu**  
- **Dễ bảo trì** và **dễ mở rộng**  
- **Tích hợp mượt mà** với JavaScript và Houdini  
- Và đặc biệt là **animation-friendly** 🎨  

> 🌿 Với `@property`, CSS không chỉ là ngôn ngữ trình bày —  
> mà còn là **ngôn ngữ mạnh mẽ, linh hoạt và có khả năng tự mô tả kiểu dữ liệu**.
