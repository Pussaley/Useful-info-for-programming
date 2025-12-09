### Docker `docker-compose.yaml`

```yaml
services:
  postgres:
    image: postgres:12.3 // or other image if need to
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=<user>
      - POSTGRES_PASSWORD=<password>
      - POSTGRES_DB=<DB>
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### PostgreSQL `init.sql`:

```sql
CREATE SCHEMA IF NOT EXISTS news_portal_schema;

SET search_path TO news_portal_schema;
```

`NewsMapper`

```java
@Component
public class NewsMapperImpl implements NewsMapper {

    @Autowired
    private UserLookupService userLookupService;
    @Autowired
    private CategoryLookUpService categoryLookUpService;

    @Override
    public News requestToNews(UpsertNewsRequest request) {
        if ( request == null ) {
            return null;
        }

        News.NewsBuilder news = News.builder();

        news.category( categoryLookUpService.toCategory( request.getCategoryId() ) );
        news.author( userLookupService.toAuthor( request.getAuthorId() ) );
        news.title( request.getTitle() );
        news.description( request.getDescription() );

        return news.build();
    }

    @Override
    public DetailedNewsResponse newsToDetailedNewsResponse(News news) {
        if ( news == null ) {
            return null;
        }

        DetailedNewsResponse detailedNewsResponse = new DetailedNewsResponse();

        detailedNewsResponse.setAuthorId( newsAuthorId( news ) );
        detailedNewsResponse.setCategoryId( newsCategoryId( news ) );
        detailedNewsResponse.setId( news.getId() );
        detailedNewsResponse.setTitle( news.getTitle() );
        detailedNewsResponse.setDescription( news.getDescription() );
        detailedNewsResponse.setComments( commentListToCommentResponseList( news.getComments() ) );

        return detailedNewsResponse;
    }

    @Override
    public LimitedNewsResponse newsToCommentsCountResponse(News news) {
        if ( news == null ) {
            return null;
        }

        LimitedNewsResponse limitedNewsResponse = new LimitedNewsResponse();

        limitedNewsResponse.setAuthorId( newsAuthorId( news ) );
        limitedNewsResponse.setCategoryId( newsCategoryId( news ) );
        limitedNewsResponse.setId( news.getId() );
        limitedNewsResponse.setTitle( news.getTitle() );
        limitedNewsResponse.setDescription( news.getDescription() );

        limitedNewsResponse.setCommentsCount( news.getComments() == null ? 0 : news.getComments().size() );

        return limitedNewsResponse;
    }

    private Long newsAuthorId(News news) {
        if ( news == null ) {
            return null;
        }
        User author = news.getAuthor();
        if ( author == null ) {
            return null;
        }
        Long id = author.getId();
        if ( id == null ) {
            return null;
        }
        return id;
    }

    private Long newsCategoryId(News news) {
        if ( news == null ) {
            return null;
        }
        Category category = news.getCategory();
        if ( category == null ) {
            return null;
        }
        Long id = category.getId();
        if ( id == null ) {
            return null;
        }
        return id;
    }

    protected CommentResponse commentToCommentResponse(Comment comment) {
        if ( comment == null ) {
            return null;
        }

        CommentResponse commentResponse = new CommentResponse();

        commentResponse.setId( comment.getId() );
        commentResponse.setText( comment.getText() );

        return commentResponse;
    }

    protected List<CommentResponse> commentListToCommentResponseList(List<Comment> list) {
        if ( list == null ) {
            return null;
        }

        List<CommentResponse> list1 = new ArrayList<CommentResponse>( list.size() );
        for ( Comment comment : list ) {
            list1.add( commentToCommentResponse( comment ) );
        }

        return list1;
    }
}
```

`CategoryMapper`

```java
@Component
public class CategoryMapperImpl implements CategoryMapper {

    @Override
    public Category requestToCategory(UpsertCategoryRequest request) {
        if ( request == null ) {
            return null;
        }

        Category.CategoryBuilder category = Category.builder();

        category.title( request.getTitle() );

        return category.build();
    }

    @Override
    public CategoryResponse categoryToResponse(Category category) {
        if ( category == null ) {
            return null;
        }

        CategoryResponse categoryResponse = new CategoryResponse();

        categoryResponse.setId( category.getId() );
        categoryResponse.setTitle( category.getTitle() );

        categoryResponse.setNews( newsMapper.newsListToListDetailedNewsResponse(category.getNewsList()) );

        return categoryResponse;
    }
}
```

`CommentMapper`

```java
@Component
public class CommentMapperImpl implements CommentMapper {

    @Autowired
    private UserLookupService userLookupService;
    @Autowired
    private NewsLookUpService newsLookUpService;

    @Override
    public Comment requestToComment(UpsertCommentRequest request) {
        if ( request == null ) {
            return null;
        }

        Comment comment = new Comment();

        comment.setAuthor( userLookupService.toAuthor( request.getAuthorId() ) );
        comment.setNews( newsLookUpService.findNews( request.getNewsId() ) );
        comment.setText( request.getText() );

        return comment;
    }

    @Override
    public CommentResponse commentToResponse(Comment comment) {
        if ( comment == null ) {
            return null;
        }

        CommentResponse commentResponse = new CommentResponse();

        commentResponse.setAuthorId( commentAuthorId( comment ) );
        commentResponse.setNewsId( commentNewsId( comment ) );
        commentResponse.setId( comment.getId() );
        commentResponse.setText( comment.getText() );

        return commentResponse;
    }

    private Long commentAuthorId(Comment comment) {
        if ( comment == null ) {
            return null;
        }
        User author = comment.getAuthor();
        if ( author == null ) {
            return null;
        }
        Long id = author.getId();
        if ( id == null ) {
            return null;
        }
        return id;
    }

    private Long commentNewsId(Comment comment) {
        if ( comment == null ) {
            return null;
        }
        News news = comment.getNews();
        if ( news == null ) {
            return null;
        }
        Long id = news.getId();
        if ( id == null ) {
            return null;
        }
        return id;
    }
}
```

`UserMapper`

```java
@Component
public class UserMapperImpl implements UserMapper {

    @Override
    public User requestToUser(UpsertUserRequest request) {
        if ( request == null ) {
            return null;
        }

        User.UserBuilder user = User.builder();

        user.firstName( request.getFirstName() );
        user.lastName( request.getLastName() );

        return user.build();
    }

    @Override
    public UserResponse userToResponse(User user) {
        if ( user == null ) {
            return null;
        }

        UserResponse userResponse = new UserResponse();

        userResponse.setId( user.getId() );
        userResponse.setFirstName( user.getFirstName() );
        userResponse.setLastName( user.getLastName() );
        Set<News> set = user.getNews();
        if ( set != null ) {
            userResponse.setNews( new LinkedHashSet<News>( set ) );
        }
        userResponse.setComments( commentListToCommentResponseSet( user.getComments() ) );

        return userResponse;
    }

    protected CommentResponse commentToCommentResponse(Comment comment) {
        if ( comment == null ) {
            return null;
        }

        CommentResponse commentResponse = new CommentResponse();

        commentResponse.setId( comment.getId() );
        commentResponse.setText( comment.getText() );

        return commentResponse;
    }

    protected Set<CommentResponse> commentListToCommentResponseSet(List<Comment> list) {
        if ( list == null ) {
            return null;
        }

        Set<CommentResponse> set = new LinkedHashSet<CommentResponse>( Math.max( (int) ( list.size() / .75f ) + 1, 16 ) );
        for ( Comment comment : list ) {
            set.add( commentToCommentResponse( comment ) );
        }

        return set;
    }
}
```
