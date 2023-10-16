---
title: "[Spring Boot] 리뷰(Review) 기능 구현"
excerpt: "리뷰(Review) 기능을 구현해보자."

categories:
  - Spring
tags:
  - [Spring Boot]

permalink: /spring/springboot-review/

toc: true
toc_sticky: true

date: 2023-02-17
last_modified_at: 2023-02-17

--- 

현재 진행하고 있는 프로젝트에 리뷰 기능이 필요하여 구현하게 되었다.<br><br>

## **리뷰 테이블 생성**
<hr />

매장(Market)에는 리뷰(Review)를 작성할 수 있다. <br>

리뷰는 매장 번호, 작성자, 내용, 생성일자, 수정일자 필드들로 구성되어 있다.

``` sql
CREATE TABLE review(
	review_id int NOT NULL AUTO_INCREMENT,
    market_id int NOT NULL,
    user_id int NOT NULL,
    content varchar(200),
    created_date datetime,
    last_modified_date datetime,
    PRIMARY KEY(review_id),
    FOREIGN KEY(market_id) REFERENCES market(market_id),
    FOREIGN KEY(user_id) REFERENCES user(user_id)
);
```

이때 리뷰 테이블의 매장 번호(market_id)와 작성자(user_id)가 외래키가 된다.<br>

리뷰를 보는 방법은 총 2가지로, 매장별 리뷰와 본인이 작성한 리뷰들을 볼 수 있게 하기 위해 각각 외래키가 필요하다.<br>

리뷰를 수정 및 삭제할 때는 해당 리뷰 번호가 있어야 가능하므로 리뷰 번호 역시 필요하다.<br><br>

## **Entity, DTO 구현**
<hr />

### **BaseTimeEntity Class**

**BaseTimeEntity** 클래스는 생성 일자, 수정 일자가 들어있는 상위 Entity이다.<br>
상속 받은 **Entity**들의 생성 일자와 수정 일자를 자동으로 관리해주는 역할이다.<br>

``` java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {
    @CreatedDate
    @Column(nullable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime lastModifiedDate;
}
```
<br>

### **Review Class**

``` java
@NoArgsConstructor
@Getter
@Entity
public class Review extends BaseTimeEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "review_id")
    private Long id;

    @Column(nullable = false, name = "market_id")
    private Long market_id;

    @Column(nullable = false, name = "user_id")
    private Long user_id;

    @Column(nullable = false)
    private String content;

    @Builder
    public Review(Long id, Long market_id, Long user_id, String content) {
        this.id = id;
        this.market_id = market_id;
        this.user_id = user_id;
        this.content = content;
    }
}
```
<br>

### **ReviewRequestDTO Class**

``` java
public class ReviewRequestDTO {
    @Getter
    @NoArgsConstructor
    public static class Create{
        private Long market_id;
        private Long user_id;
        private String content;

        @Builder
        public Create(final Long market_id, final Long user_id, final String content) {
            this.market_id = market_id;
            this.user_id = user_id;
            this.content = content;
        }

        public Review toEntity(){
            return Review.builder()
                    .market_id(market_id)
                    .user_id(user_id)
                    .content(content)
                    .build();
        }
    }
}
```
<br>

### **ReviewResponseDTO Class**

``` java
public class ReviewResponseDTO {
    @Getter
    @Builder
    public static class Info{
        private Long review_id;
        private Long market_id;
        private Long user_id;
        private String content;
        @JsonFormat(shape= JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm:ss")
        private LocalDateTime createdDate;
        @JsonFormat(shape= JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm:ss")
        private LocalDateTime lastModifiedDate;

        public static ReviewResponseDTO.Info of(Review review){
            return Info.builder()
                    .review_id(review.getId())
                    .market_id(review.getMarket_id())
                    .user_id(review.getUser_id())
                    .content(review.getContent())
                    .createdDate(review.getCreatedDate())
                    .lastModifiedDate(review.getLastModifiedDate())
                    .build();
        }
    }
}
```

<br>

## **Service 구현**
<hr />

``` java
@Service
@RequiredArgsConstructor
public class ReviewService {
    private final ReviewRepository reviewRepository;

    public ReviewResponseDTO.Info createReview(ReviewRequestDTO.Create createDTO){
        Review review = createDTO.toEntity();

        return ReviewResponseDTO.Info.of(reviewRepository.save(review));
    }

    public ReviewResponseDTO.Info findById(Long id){
        Review review = reviewRepository.findById(id)
                .orElseThrow(() ->
                        new IllegalArgumentException("해당 리뷰가 존재하지 않습니다. " + id));

        return ReviewResponseDTO.Info.of(review);
    }

    @Transactional
    public ReviewResponseDTO.Info editReview(ReviewRequestDTO.Edit editDTO){
        Review review = reviewRepository.findById(editDTO.getReview_id())
                .orElseThrow(() ->
                        new IllegalArgumentException("해당 리뷰가 존재하지 않습니다. " + editDTO.getReview_id()));

        review.update(editDTO.getContent());

        return ReviewResponseDTO.Info.of(review);
    }

    @Transactional
    public void deleteReview(Long review_id){
        if(!reviewRepository.existsById(review_id))
            throw new IllegalArgumentException("해당 리뷰가 존재하지 않습니다. " + review_id);

        reviewRepository.deleteById(review_id);
    }
}
```

현재까지는 Review Entity에 대한 CRUD 기능을 구현했다.<br>

추후에 현재 로그인한 사용자를 체크하여 사용자가 작성한 리뷰에 대해서만 수정 및 삭제가 가능하도록 추가할 예정이다.<br><br>

## **Controller 구현**
<hr />

``` java
@RequiredArgsConstructor
@RequestMapping("/review")
public class ReviewController {
    private final ReviewService reviewService;

    @PostMapping("/create")
    private ReviewResponseDTO.Info createReview(@RequestBody ReviewRequestDTO.Create createDTO){
        return reviewService.createReview(createDTO);
    }

    @GetMapping("/findById")
    private ReviewResponseDTO.Info findById(Long review_id){
        return reviewService.findById(review_id);
    }

    @PatchMapping("/edit")
    private ReviewResponseDTO.Info editReview(@RequestBody ReviewRequestDTO.Edit editDTO){
        return reviewService.editReview(editDTO);
    }

    @DeleteMapping("/delete")
    private void deleteReview(Long review_id){
        reviewService.deleteReview(review_id);
    }
}
```
<br>

**마치며**<br>

기능을 구현할 때마다 단위 테스트를 작성하려 했지만, 테스트 코드를 작성하는데 있어서 아직 미숙하여 모든 기능에 대한 테스트 코드를 작성하지 못한 점이 아쉬웠다!<br>
