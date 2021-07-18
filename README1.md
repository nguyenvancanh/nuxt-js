# Sử dụng Nuxt Content tạo một blog

Vẫn nằm trong series vừa học vừa làm, hôm nay tôi xin giới thiệu tới quý bạn đọc một module vồ cùng hữu ích của Nuxt  [Nuxt content](https://content.nuxtjs.org/), đây là một module vô cùng hữu ích để bạn có thể tạo blog, tạo trang tài liệu, hay đơn giản là thêm nội dung vào trang web có sẵn của bạn. Trong bài viết này, tôi sẽ cố gắng giới thiệu hầu hết tính năng của module này có, và cũng thử xây dựng một blog dựa trên nó. Nếu bạn quan tâm thì hãy đi tới cuối bài viết nhé

## Chuẩn bị

Trước hết, để sử dụng được module này thì đương nhiên chúng ta cần phải cài đặt nó đã. Câu lệnh cài đặt như sau

```
// với yarn
yarn add @nuxt/content
// Với npm 
npm install @nuxt/content
```

Sau khi cài đặt xong, bạn cần khai báo nó trong file nuxt.config

```
export default {
  modules: ['@nuxt/content']
}
```

Tiếp theo, tạo mới thư mục cũng như nhưng file cần thiết

```
mkdir content
```
Content module sẽ đọc các file trong thư mục content/ mà chúng ta vừa tạo ra ở trên, tiếp tới hãy tạo thư mục articles/, trong đây sẽ chứa toàn bộ bài viết vủa blog mà chúng ta sắp xây dựng

```
mkdir content/articles
```

Có 1 điểm thú vị của module này là nó có thể đọc được các file với định dạng markdown, csv, yaml, json, json5 hoặc xml. Hãy thử bắt đầu với một file markdown nhé

```
touch content/articles/my-first-blog-post.md
```

Bây giờ chúng ta có thể thêm title và text cho trang của mình

```
# My first blog post

Welcome to my first blog post using content module
```
Công việc tiếp theo là cẩn hiển thị nội dung chúng ta vừa thêm lên trên web, để làm được việc này, chúng ta cần khai báo router cho trang web của mình, tạo file _slug.vue trong thư mục blog của mình, và thêm nội dung code như sau

```
touch pages/blog/_slug.vue
```

```
<script>
  export default {
    async asyncData({ $content, params }) {
      // fetch our article here
    }
  }
</script>
```

ở đây chúng ta dùng hàm asyncData để cập nhật content của bài viết, sau đó return lại cho phía client. Giải thích thì hơi dài dòng, cụ thể code như sau

```
<script>
  export default {
    async asyncData({ $content, params }) {
      const article = await $content('articles', params.slug).fetch()

      return { article }
    }
  }
</script>
```

Sau khi chuuẩn bị module hoàn tất,công việc còn lại của chúng ta là gọi content trong template là hoàn tất

```
// pages/blog/_slug.vue

<template>
  <article>
    <nuxt-content :document="article" />
  </article>
</template>
```

Giờ hãy truy cập vào link [http://localhost:3000/blog/my-first-blog-post ](http://localhost:3000/blog/my-first-blog-post ) bạn sẽ thấy content trong file được hiện lên trên màn hình

## Default Injected variables

Nuxt conent module cho phép chúng ta sử dụng những biến defautl, đây là danh sách các bieetns mà bạn có thể sử dụng

-body: body text
-dir: directory
-extension: file extension (.md in this example)
-path: the file path
-slug: the file slug
-toc: an array containing our table of contents
-createdAt: the file creation date
- updatedAt: the date of the last file update

Tất cả các biến trên chúng ta có thể dễ dàng truy cập vào từ biến articles chúng ta đã tạo ở bên trên, nếu các bạn chưa tin, hãy thử in biến article ra bằng lệnh

```
<pre> {{ article }} </pre>
```
Output thu được là: 

```
"dir": "/articles",
"path": "/articles/my-first-blog-post",
"extension": ".md",
"slug": "my-first-blog-post",
"createdAt": "2020-06-22T10:58:51.640Z",
"updatedAt": "2020-06-22T10:59:27.863Z"
```

Có nghĩa là trong template của bạn, có có thể tùy ý sử dụng bất kỳ variable nào bằng cách

```
pages/blog/_slug.vue
<p>Post last updated: {{ article.updatedAt }}</p>
```

Có một điều đó là ngày tháng format chưa được thân thiện với người dùng cho lắm đúng k, đơn giản thôi, format lại 1 chút là được

```
pages/blog/_slug.vue
methods: {
    formatDate(date) {
      const options = { year: 'numeric', month: 'long', day: 'numeric' }
      return new Date(date).toLocaleDateString('en', options)
    }
 }
```
khai báo method formatDate với tham số truyền vào có định dạng date, cách sử dụng thì như sau

```
pages/blog/_slug.vue
<p>Article last updated: {{ formatDate(article.updatedAt) }}</p>
```

## Custom Inject variable

Trong file .md của mình, thay vì truyền thẳng text xuống thì chúng ta thay đổi lại như sau

```
// content/articles/my-first-blog-post.md

---
title: My first Blog Post
description: Learning how to use @nuxt/content to create a blog
img: first-blog-post.jpg
alt: my first blog post
---
```
Tất nhiên, khi sửa lại như trên thì file _slug.vue của chúng ta cũng cần phải cập nhật

```
<template>
  <article>
    <h1>{{ article.title }}</h1>
    <p>{{ article.description }}</p>
    <img :src="article.img" :alt="article.alt" />
    <p>Article last updated: {{ formatDate(article.updatedAt) }}</p>

    <nuxt-content :document="article" />
  </article>
</template>
```

Công việc tiếp theo là cần phải thêm style cho bắt mắt người xem

```
// pages/blog/_slug.vue
<style>
  .nuxt-content h2 {
    font-weight: bold;
    font-size: 28px;
  }
  .nuxt-content h3 {
    font-weight: bold;
    font-size: 22px;
  }
  .nuxt-content p {
    margin-bottom: 20px;
  }
</style>
```

Làm cho heading nổi bật hơn so với content, thì thêm class

```
.icon.icon-link {
  background-image: url('~assets/svg/icon-hashtag.svg');
  display: inline-block;
  width: 20px;
  height: 20px;
  background-size: 20px 20px;
}
```

Sử dụng toc để tạo 1 link khi click vào sẽ di chuyển tới một section mà bạn mong muốn. Trước hết,khai báo danh sách các header như sau

```
// content/articles/my-first-blog-post.md

## This is a heading

This is some more info

### This is a sub heading

This is some more info

### This is another sub heading

This is some more info

## This is another heading

This is some more info
```

Tiếp đến, trong file _slg.vue, thêm đoạn code

```
<nav>
  <ul>
    <li v-for="link of article.toc" :key="link.id">
      <NuxtLink :to="`#${link.id}`">{{ link.text }}</NuxtLink>
    </li>
  </ul>
</nav>
```
Bây giờ thì toc đã hoạt động rồi đó, khi bạn click vào link, thì giao diện sẽ ngay lập tức di chuyển màn hình của bạn tới header tương ứng. Content module sẽ tự động thêm id và link vào từng header tương ứng bạn đã khai báo, bạn có thể dễ dàng kiểm tra với việc inspect những thẻ h2 đang được in ra trên màn hình. Và dễ nhiên, bạn có thể dễ dàng custom lại các class cho từng header của mình bằng cách 

```
:class="{ 'py-2': link.depth === 2, 'ml-2 pb-2': link.depth === 3 }"
```

## Sử dụng HTML trong file markdown

Đôi khi bạn mong muốn có thể sử dụng HTML được trong file markdown. Điều này cũng k quá khó khăn, hãy thử thêm 1 thẻ div kèm theo đó là các class như sau

```
// content/articles/my-first-blog-post.md

<div class="p-4 mb-4 text-white bg-blue-500">
  This is HTML inside markdown that has a class of note
</div>
```
Vô cùng đơn giản đúng k nào.

