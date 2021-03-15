
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


예를 보자.

    val spannableStringBuilder = SpannableStringBuilder("Android")
    spannableStringBuilder.setSpan(
        ForegroundColorSpan(Color.RED),
        1, // start
        4, // end
        Spannable.SPAN_INCLUSIVE_INCLUSIVE
    )
    spannableStringBuilder.insert(4, "1")
    spannableStringBuilder.insert(1, "1")
    tvMessage.text = spannableStringBuilder

플래그에 따른 결과를 보자.

![span-flag-example-9530cea2d13fe0ec](https://user-images.githubusercontent.com/19990905/111117332-bacf1480-85aa-11eb-9f65-24e2f8c14fd3.png)


</br>


## BulletSpan

bullet list를 만드는 Span이다.

예를 보자.

    // function to covert a list into bullet list
    fun convertToBulletList(stringList: List<String>): CharSequence {
        val spannableStringBuilder = SpannableStringBuilder("Learn Android from\n")
        stringList.forEachIndexed { index, text ->
            val line: CharSequence = text + if (index < stringList.size - 1) "\n" else ""
            val spannable: Spannable = SpannableString(line)
            spannable.setSpan(
                BulletSpan(15, Color.RED),
                0,
                spannable.length,
                Spanned.SPAN_INCLUSIVE_EXCLUSIVE
            )
            spannableStringBuilder.append(spannable)
        }
        return spannableStringBuilder
    }
    val androidResourceList = listOf("MindOrks Course", "MindOrks Blog", "MindOrks OpenSource", "MindOrks YouTube")
    tvMessage.text = convertToBulletList(androidResourceList)
    
결과를 보자.
    
![image](https://user-images.githubusercontent.com/19990905/111117596-139ead00-85ab-11eb-8bfc-eb061304504c.png)



참조 : 
https://blog.mindorks.com/spannable-string-text-styling-with-spans
