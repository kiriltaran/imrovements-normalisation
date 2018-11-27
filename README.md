# Data normalisation

## What is it?

It is the process of reorganizing data, that meets two basic requirements:

- there is no redundancy of data (all data is stored in only one place)
- data dependencies are logical (all related data items are stored together).

## Why should do it?

- Decreased memory usage of the app
- Easier to work with the nested objects to retrieve a nested object and faster to access
- No nasty code to look up values in the nested objects
- Because each item is only defined in one place, we don’t have to try to make changes in multiple places if that item is updated.
- The logic for retrieving or updating a given item is now fairly simple and consistent. Given an item’s type and its ID, we can directly look it up in a couple simple steps, without having to dig through other objects to find it.
- Since each data type is separated, an update nested child field of the object would only require new copies of the field value. This will generally mean fewer number of the component re-renders because their data has changed. In contrast, updating a data in the original nested shape would have required updating the whole parent(root) object and likely have caused all of the child components in the UI to re-render themselves.

## How to do it?

**Main rules**:

- Each type of data gets its own "table" in the state.
- Each "data table" should store the individual items in an object, with the IDs of the items as keys and the items themselves as the values.
- Any references to individual items should be done by storing the item's ID.
- Arrays of IDs should be used to indicate order.

**Bad**

```javascript
state: () => ({
  posts: [
    {
      id: 'postid1',
      content: 'Lorem ipsum dolor sit amet...',
      author: {
        id: 'authorid1',
        nickname: 'Evan Abramov'
      },
      comments: [
        {
          id: 'commentid1',
          content: 'Comment text',
          author: {
            id: 'authorid2',
            nickname: 'Bill Torvalds'
          }
        }
      ]
    },
    {
      id: 'postid2',
      content: 'consectetur adipiscing elit...',
      author: {
        id: 'authorid2',
        nickname: 'Bill Torvalds'
      },
      comments: []
    }
  ]
});
```
**Good**

```javascript
state: () => ({
  posts: {
    postid1: {
      id: 'postid1',
      content: 'Lorem ipsum dolor sit amet...',
      authorId: 'authorid1',
      commentIds: ['commentid1']
    },
    postid2: {
      id: 'postid2',
      content: 'consectetur adipiscing elit...',
      authorId: 'authorid2',
      commentIds: []
    }
  },
  authors: {
    authorid1: {
      id: 'authorid1',
      nickname: 'Evan Abramov'
    },
    authorid2: {
      id: 'authorid2',
      nickname: 'Bill Torvalds'
    }
  },
  comments: [
    {
      id: 'commentid1',
      content: 'Comment text',
      authorId: 'authorid2'
    }
  ]
})
```

## How to automate it?
We have two popular packages to normalise data in our JavaScript applications

- [JSON API Normalizer](https://github.com/yury-dymov/json-api-normalizer) : *transforms data in JSON API standard([JSON API Specification](https://jsonapi.org/)) to normalized data without configuration*

- [Normalizr](https://github.com/paularmstrong/normalizr) : *uses custom schemas definition to create normalized data.*

**Normalizr example**

```javascript
// Initial denormalized posts state
const posts = [
  {
    id: 1,
    content: "Lorem ipsum dolor sit amet...",
    author: {
      id: 3,
      nickname: "Evan Abramov"
    },
    comments: [
      {
        id: 5,
        content: "Comment text",
        author: {
          id: 4,
          nickname: "Bill Torvalds"
        }
      }
    ]
  },
  {
    id: 2,
    content: "consectetur adipiscing elit...",
    author: {
      id: 4,
      nickname: "Bill Torvalds"
    },
    comments: []
  }
];

// Schema descriptions
const authorSchema = new schema.Entity("authors");

const commentSchema = new schema.Entity("comments", {
  author: authorSchema
});

const postSchema = new schema.Entity("posts", {
  author: authorSchema,
  comments: [commentSchema]
});

const postsSchema = new schema.Array(postSchema);

// Normalizing post by defined schema
const normalizedPosts = normalize(posts, postsSchema);

console.log(normalizedPosts)
/*
{
  "entities": {
    "authors": {
      "3": {
        "id": 3,
        "nickname": "Evan Abramov"
      },
      "4": {
        "id": 4,
        "nickname": "Bill Torvalds"
      }
    },
    "comments": {
      "5": {
        "id": 5,
        "content": "Comment text",
        "author": 4
      }
    },
    "posts": {
      "1": {
        "id": 1,
        "content": "Lorem ipsum dolor sit amet...",
        "author": 3,
        "comments": [
          5
        ]
      },
      "2": {
        "id": 2,
        "content": "consectetur adipiscing elit...",
        "author": 4,
        "comments": []
      }
    }
  },
  "result": [
    1,
    2
  ]
}
*/
```

## Conclusion

### Pros

- updating data is very easy
- easily accessing to all entities
- no data duplication

### Cons

- accessing few entities item in another entity requires passing an object with all items
- in small apps without a lot of data duplication, normalizing data may not be necessary
