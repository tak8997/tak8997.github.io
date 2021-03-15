
# Spannable String


## What are Spans?

SpannedString : 생성된 후 글자, markup을 변경할 필요가 없을 때 사용된다.
SpannableString : 생성된 후 글자의 변경이 필요 없을 때, markup 변경이 필요할 때 사용된다.
SpannableStringBuilder : 생성된 후 글자, markup을 둘 다 변경할 필요가 있을 때 사용된다.


<br/>


## Spannable Flags

자주 쓰이는 4개.
 - SPAN_EXCLUSIVE_EXCLUSIVE
 - SPAN_EXCLUSIVE_INCLUSIVE
 - SPAN_INCLUSIVE_EXCLUSIVE
 - SPAN_INCLUSIVE_INCLUSIVE

이 플래그들은 Span에 시작 또는 끝 위치에 삽입 된 텍스트를 포함해야하는지 여부를 Span에 알리는 데 사용됨. 
시작, 끝 위치에 삽입 된 텍스트에 대해서도 Span을 적용할지를 말하는 것임.
 
Inclusive는 Span을 적용해라.
Exclusive는 Span을 제거해라.


예를 
