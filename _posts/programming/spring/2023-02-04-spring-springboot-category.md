---
title: "[Spring Boot] JPA를 사용한 카테고리(Infinite Depth) 구현"
excerpt: "JPA를 사용하여 하위 메뉴까지 가져올 수 있는 무한 뎁스 카테고리 로직을 구현해보자."

categories:
  - Spring
tags:
  - [Spring Boot]

permalink: /spring/springboot-category/

toc: true
toc_sticky: true

date: 2023-02-04
last_modified_at: 2023-02-05

--- 

현재 개발하고 있는 **Jangbogo** Web 사이트에 카테고리 기능이 필요하여 구현하게 되었다.<br>
**depth**를 설정하여 하위 메뉴의 데이터들까지 가져올 수 있는 카테고리 기능을 구현해보자!<br><br>

## **요구조건 및 개발 순서**
<hr />

**1. Entity 구현**<br>
* 하나의 테이블에서 구현해야 하기 때문에 <span style="color:red">**Self Join**</span>을 사용하여 무한 뎁스 구현
* level 필드를 구현하여 depth에 따라 level별 분류하도록 구현
<br>

**2. Repository 구현**<br>
* JPA 활용을 위해 JpaRepository를 extends
<br>

**3. DTO 구현**<br>
* Entity는 데이터를 생성하고 DB하고만 소통하게 하기 위해 DTO 구현
* DTO는 비지니스 로직에서 사용하기 위해 구현
<br>

**4. Service 구현**<br>
* createCategory 메서드
  * `CategoryRequestDTO`를 매개변수로 받아 카테고리 생성
* getCategoryByName 메서드
  * 카테고리의 `name`으로 검색하여 해당 카테고리와 하위 카테고리 모두 반환
<br>

**5. Service 메서드별 테스트 구현**<br>
* given, when, then 순으로 테스트 코드 구현
<br>

**6. Controller 구현**<br>
* api 통신을 통해 각각의 반환 로직 구현
<br><br>

## **Category Entity 구현**
<hr />

``` java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "category_id")
    private Long id;

    @Column(name = "name")
    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private Category parent;

    @Column(name = "depth")
    private Long depth;

    @OneToMany(mappedBy = "parent")
    private List<Category> children = new ArrayList<>();
}
```

**1. @ManyToOne (fetch = FetchType.LAZY)**
* 하위 카테고리 입장에서는 상위 카테고리에 속해 있는 하위 카테고리가 N개일 수 있으므로 **N:1** 관계이기 때문에 `@ManyToOne` 어노테이션을 붙여준다.
* 카테고리 정보를 반환할 때 부모 카테고리의 정보는 필요 없으므로 <span style="color:red">**지연 로딩(FetchType.LAZY)**</span>으로 설정해준다.
<br>

**2. OneToMany(mappedBy = "parent")**
* 상위 카테고리 입장에서는 하위 카테고리가 N개일 수 있으므로 **1:N** 관계이기 때문에 `@OneToMany` 어노테이션을 붙여준다.
* mappedBy 속성을 통해 parent를 연관 관계의 <span style="color:red">**owner**</span>로 설정해준다.
<br><br>

## **Category Repository 구현**
<hr />

``` java
public interface CategoryRepository extends JpaRepository<Category, Long> {

    Optional<Category> findById(Long category_id);
    Optional<Category> findByName(String name);

    Boolean existsByName(String name);
}
```

**1. JpaRepository 상속**<br>

**2. findById, findByName**<br>
* category_id와 name으로 DB를 탐색하여 해당 카테고리를 반환하는 기능
  
**3. existsByName**<br>
* name으로 DB를 탐색하여 해당 카테고리의 존재 여부를 반환하는 기능

<br>

## **Category DTO 구현**
<hr />

``` java
@Getter
@Setter
@AllArgsConstructor
public class CategoryRequestDTO {
    private String name;
    private String parentName;
}
```
``` java
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class CategoryResponseDTO {
    private Long id;
    private String name;
    private String parentName;
    private Long depth;
    private List<CategoryResponseDTO> children;

    public static CategoryResponseDTO of(Category category){
        return CategoryResponseDTO.builder()
                .id(category.getId())
                .name(category.getName())
                .parentName(category.getParent() == null ? null : category.getParent().getName())
                .depth(category.getDepth())
                .children(category.getChildren() == null ? null :
                        category.getChildren().stream().map(CategoryResponseDTO::of).collect(Collectors.toList()))
                .build();
    }
}
```

**1. CategoryRequestDTO**<br>
* 카테고리 생성할 때 필요한 데이터를 가진 DTO

**2. CategoryResponseDTO**<br>
* 카테고리 정보를 반환할 때 필요한 데이터를 가진 DTO
* 카테고리를 매개변수로 받아 CategoryResponseDTO를 생성해 반환하는 `of` 정적 메서드
<br><br>

## **Category Service 구현**
<hr />

### **saveCategory 구현**

``` java
@Service
@RequiredArgsConstructor
@Transactional
public class CategoryService {
    private final CategoryRepository categoryRepository;

    public Long saveCategory(CategoryRequestDTO categoryRequestDTO){
        Category category;

        // 상위 카테고리가 없다면 대분류로 등록
        if(categoryRequestDTO.getParentName() == null){
            Category rootCategory = categoryRepository.findByName("ROOT")
                    .orElseGet(() ->
                            Category.builder()
                                    .name("ROOT")
                                    .depth(0l)
                                    .build()
                    );

            // ROOT 카테고리가 존재하지 않다면 생성
            if(!categoryRepository.existsByName("ROOT")){
                categoryRepository.save(rootCategory);
            }

            category = Category.builder()
                    .name(categoryRequestDTO.getName())
                    .parent(rootCategory)
                    .depth(rootCategory.getDepth() + 1)
                    .build();
        }else{
            String parentName = categoryRequestDTO.getParentName();
            Category parent = categoryRepository.findByName(parentName)
                    .orElseThrow(() -> new IllegalArgumentException("부모 카테고리 없음 예외"));

            category = Category.builder()
                    .name(categoryRequestDTO.getName())
                    .parent(parent)
                    .depth(parent.getDepth() + 1)
                    .build();

            parent.getChildren().add(category);
        }

        return categoryRepository.save(category).getId();
    }
}
```

**1. ROOT 카테고리**<br>
* 모든 카테고리 정보를 반환해야 할 수 있으므로 최상위 카테고리가 존재해야 한다.
* 상위 카테고리 정보가 없는 `CategoryRequestDTO`가 매개변수로 들어왔을 때 DB에서 ROOT 카테고리의 존재 여부를 확인 후, 없다면 생성해준다.

**2. orElseGet, orElseThrow**<br>
* ROOT가 존재하지 않을 경우 `Optional` 타입으로 반환하는 Repository의 메서드를 `orElseGet`을 사용하여 ROOT 카테고리를 생성하도록 구현
* 하위 카테고리를 생성하는데 `CategoryRequestDTO` 데이터의 상위 카테고리 정보가 DB에 존재하지 않다면 `orElseThrow`를 사용하여 예외처리 하도록 구현

<br>

### **getCategoryByName 구현**

``` java
public CategoryResponseDTO getCategoryByName(String name){
    Category category = categoryRepository.findByName(name)
            .orElseThrow(() -> new IllegalArgumentException("해당 카테고리는 존재하지 않습니다."));

    CategoryResponseDTO categoryResponseDTO = CategoryResponseDTO.of(category);

    return categoryResponseDTO;
}
```

**1. 카테고리 id 값으로 DB에서 카테고리를 찾아 반환**<br>

**2. 카테고리 name 값으로 DB에서 카테고리를 찾아 반환**<br><br>

## **Category Service Test 구현**
<hr />  

``` java
@SpringBootTest
@Transactional
class CategoryServiceTest {
    @Autowired
    CategoryService categoryService;

    // CreateId
    private CategoryRequestDTO categoryRequestDTO(String name, String parent) {
        CategoryRequestDTO categoryRequestDTO = new CategoryRequestDTO(name, parent);
        
        return categoryRequestDTO;
    }

    // Find Category
    private CategoryResponseDTO findCategory(Long id){
        return categoryService.findCategory(id);
    }

    @Test
    void 카테코리_생성_테스트() {
        // given
        CategoryRequestDTO categoryRequestDTO = categoryRequestDTO("TestName", null);
        Long saveId = categoryService.createCategory(categoryRequestDTO);

        // when
        CategoryResponseDTO categoryResponseDTO = findCategory(saveId);

        // then
        assertThat(categoryResponseDTO.getName()).isEqualTo("TestName");
    }
}
```

<hr />  
참고자료<br>
<a href="https://velog.io/@joshuara7235/JPA-%EC%82%AC%EC%9A%A9%ED%95%9C-%EC%B9%B4%ED%85%8C%EA%B3%A0%EB%A6%AC-%EA%B5%AC%ED%98%84-infinite-depth-01">https://velog.io/@joshuara7235/</a><br>