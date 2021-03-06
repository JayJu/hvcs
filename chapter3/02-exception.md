# 예외처리

1. REST 예외처리 절차
   1. 예외 클래스 생성
      ```java
      public class CustomParameterizedException extends RuntimeException {
      
         private static final long serialVersionUID = 1L;
         private static final String PARAM = "param";
         private final String message;
         private final Map<String, String> paramMap = new HashMap<>();
         public CustomParameterizedException(String message, String... params) {
            super(message);
            this.message = message;
            if (params != null && params.length > 0) {
               for (int i = 0; i < params.length; i++) {
                  paramMap.put(PARAM + i, params[i]);
               }
            }
         }
      ..
      ```
   2. @ControllerAdvice 로 전역 예외처리 클래스 생성
   ``` java
   @ControllerAdvice
   public class ExceptionTranslator {
      
         @ExceptionHandler(CustomParameterizedException.class)
         @ResponseStatus(HttpStatus.BAD_REQUEST)
         @ResponseBody
         public ParameterizedErrorVM processParameterizedValidationError(CustomParameterizedException ex) {
               return ex.getErrorVM();
            }
      ..
   ```
   3. Anguler json에 예외 메시지 키/값 정의
   ``` javascript
   {
         "hvcsApp": {
            "conference" : {
               "errorMessage": {
                  "notExistHostId": "Please check that there is host id' value"
               }
      ..
   ```
   4. @Controller 에서 메시지 키와 함께 예외 throw
   ``` java
   @RestController
   @RequestMapping("/api")
   public class ConferenceResource {
         @PostMapping("/conferences")
         @Timed
         public ResponseEntity<Conference> createConference(@Valid @RequestBody Conference conference) throws URISyntaxException, IOException {
            log.debug("REST request to save Conference : {}", conference);
            log.debug("conferenceFile :::: " + conference.getConferenceFiles());
            
            String hostId = "";
            Optional<User> user = userService.getUserByLogin(SecurityUtils.getCurrentUserLogin());
            hostId = user.isPresent() ? user.get().getLogin() : "";
            
            String errorMessage = "hvcsApp.conference.errorMessage.notExistHostId";
            if ("".equals(hostId)) {
            throw new CustomParameterizedException(errorMessage, "save conference");
         }
   ```
   5. @ControllerAdvice 에서 @ExceptionHandler 로 해당 예외 찾아서 처리 후 client에게 response 전달
   6. client 에서 response  의 message key 로 value 찾아 화면에 출력



