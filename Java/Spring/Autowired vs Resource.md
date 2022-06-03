# Autowired vs Resource



`@Autowired`와 `@Resource`는 둘다 필드에 빈을 주입 받을때 사용한다.



차이점은

- `@Autowired`는 필드의 **타입**을 기준으로 빈을 찾는다.
- `@Resource`는 필드의 **이름**을 기준으로 빈을 찾는다.