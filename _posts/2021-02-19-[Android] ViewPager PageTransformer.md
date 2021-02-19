# ViewPager PageTransformer

부스트코스 프로젝트4를 하면서 뭐에 대해 글을 써볼까 고민하다 뷰 페이저의 PageTransformer에 대해서 써보기로 했습니다.
그래서 PageTransformer가 뭐냐면 뷰페이저 페이지 전환에 다양한 효과를 주기 위해 사용되는 클래스 입니다. 

먼저, PageTransformer를 적용시키기 전에, 현재 보여지는 뷰페이저 화면의 양 싸이드 페이지를 부분적으로 보여지도록 할 것입니다.
적용한 코드를 보도록 하겠습니다.

    binding.viewpager.setClipToPadding(false);
    binding.viewpager.setPadding(100, 0, 100, 0);
    
이렇게 하면 양 싸이드 페이지가 부분적으로 보여지게 됩니다. 여기서 한 가지 짚어볼 것은 setClipToPadding 정도 입니다. ViewGroup하위의 
Child들에게 Padding 값이 먹도록 해줍니다.
그 다음으로는 setPageTransformer메소드에 ViewPager.PageTransformer의 구현체를 작성함으로써 원하는 효과를 넣어줄 수 있습니다.

    binding.viewpager.setTransformer(false, new ViewPager.PageTransformer() {
        @Override
        public void transformPage(@NonNull View page, float position) {
            if (position <= 0) {
                page.setAlpha(0.3F);
                page.setScaleY(0.8F);
            } else if (position <= 1) {
                page.setAlpha(1F);
                page.setScaleY(1F);
            } else {
                page.setAlpha(0.3F);
                page.setScaleY(0.8F);
            }
        }
    }
    
요런식으로 작성되어 있습니다. 설명을 해 보자면ㅡ 여기서는 없지만, 
페이지가 왼쪽으로 완전히 이동한 경우, position < -1 
페이지가 왼쪽으로 이동 되어가고 있는 경우 position <= 0
페이지가 새롭게 나타나고 있는 경우 position <= 1 입니다.
나머지 페이지가 오른쪽으로 이동하는 경우 position > 1 입니다.

여기서는 alpha, scale 값만의 효과를 주었습니다. 이런식으로 원하는 효과를 주어서 뷰페이저 페이지 전환에 다양한 효과를 줄 수 있습니다.


