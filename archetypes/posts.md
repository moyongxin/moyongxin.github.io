+++
date = '{{ .Date }}'
draft = true
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
slug = '{{ .File.ContentBaseName }}'
author = 'qwertyuiop'
tags = []
keywords = []
readingTime = true
showFullContent = false
hideComments = false
+++

<!-- CC-BY-SA 4.0 -->
&copy; {{ (.Date | time.AsTime).Year }} [moyongxin](https://github.com/moyongxin). This website's content is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0).
{ style="color: color-mix(in srgb,var(--foreground) 65%,transparent); margin-bottom: 0px;" }
