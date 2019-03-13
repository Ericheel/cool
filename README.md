# cool

## 富文本替换指定字符串，似乎有用，先记下来

stringBody.replaceAll("(?i)(\\b" + "destination" + "\\b)(?!([^<]+)?>)", "replaceText");
