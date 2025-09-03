+++
date = '{{ .Date }}'
draft = true
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
slug = '{{ replace .File.ContentBaseName "-" " " }}'
author = 'qwertyuiop'
cover = ""
tags = []
keywords = []
readingTime = true
showFullContent = false
hideComments = false
+++

---
<!-- CC-BY-SA 4.0 -->
"[{{ replace .File.ContentBaseName "-" " " | title }}]({{ .Site.BaseURL }}{{ .File.Path | path.Clean }})" &copy; {{ (.Date | time.AsTime).Year }} by [moyongxin](https://github.com/moyongxin) is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0)
{ style="color: color-mix(in srgb,var(--foreground) 65%,transparent); margin-bottom: 0px;" }
