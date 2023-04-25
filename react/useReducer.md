# useReducer

## Post demo

`useState`

```tsx
const Post = (): JSX.Element => {
const [loading, setLoading] = useState<boolean>(false);
const [error, setError] = useState<boolean>(false);
const [post, setPost] = useState<{ [key: string]: any }>({});
const handleFetch = (): void => {
  setLoading(true);
  setError(false);
  fetch("https://jsonplaceholder.typicode.com/posts/1")
    .then((res) => res.json())
    .then((data) => {
      setLoading(false);
      setPost(data);
    })
    .catch((err) => {
      setLoading(false);
      setError(true);
    });
};

 return (
    <div>
      <button onClick={handleFetch}>
        {loading ? "Wait..." : "Fetch the post"}
      </button>
      <p>{post?.title}</p>
      <span>{error && "Something went wrong!"}</span>
    </div>
  );
}

```

`useReducer`

```tsx
// postActionTypes.ts
export const ACTION_TYPES = {
  FETCH_START: "FETCH_START",
  FETCH_SUCCESS: "FETCH_SUCCESS",
  FETCH_ERROR: "FETCH_ERROR",
};
// postReducer.tsx
import { ACTION_TYPES } from "./postActionTypes";

export interface PostState {
  loading: boolean;
  post: { [key: string]: any };
  error: boolean;
}

export const INITIAL_STATE: PostState = {
  loading: false,
  post: {},
  error: false,
};

export const postReducer = (state: PostState, action: { type: string, payload?: { [key: string]: any }}) => {
  switch (action.type) {
    case ACTION_TYPES.FETCH_START:
      return {
        loading: true,
        error: false,
        post: {},
      };
    case ACTION_TYPES.FETCH_SUCCESS:
      return {
        ...state,
        loading: false,
        post: action.payload,
      };
    case ACTION_TYPES.FETCH_ERROR:
      return {
        error: true,
        loading: false,
        post: {},
      };
    default:
      return state;
  }
};
  // Post.tsx
import { useReducer, useState } from "react";
import { ACTION_TYPES } from "./postActionTypes";
import { INITIAL_STATE, postReducer } from "./postReducer";
const Post = (): JSX.Element  => {
  const [state, dispatch] = useReducer(postReducer, INITIAL_STATE);
  const handleFetch = (): void => {
    dispatch({ type: ACTION_TYPES.FETCH_START });
    fetch("https://jsonplaceholder.typicode.com/posts/1")
      .then((res) => {
        return res.json();
      })
      .then((data) => {
        dispatch({ type: ACTION_TYPES.FETCH_SUCCESS, payload: data });
      })
      .catch((err) => {
        dispatch({ type: ACTION_TYPES.FETCH_ERROR });
      });
  };
  return (
    <div>
      <button onClick={handleFetch}>
        {loading ? "Wait..." : "Fetch the post"}
      </button>
      <p>{post?.title}</p>
      <span>{error && "Something went wrong!"}</span>
    </div>
  );
}


```

## Form demo

`useState`

```tsx
//demo2 Form
import { useState, useRef } from "react";

interface Product {
  title: string;
  desc: string;
  price: number;
  category: string;
  tags: string[];
  images: {
    sm: string;
    md: string;
    lg: string;
  };
  quantity: number;
}

const MyComponent = (): JSX.Element => {
  const [product, setProduct] = useState<Product>({
    title: "",
    desc: "",
    price: 0,
    category: "",
    tags: [],
    images: {
      sm: "",
      md: "",
      lg: "",
    },
    quantity: 0,
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>): void => {
    setProduct((prev) => ({ ...prev, [e.target.name]: e.target.value }));
  };

  const tagRef = useRef<HTMLInputElement>(null);

  const handleTags = (): void => {
    const tags = tagRef.current?.value.split(",") || [];
    tags.forEach((tag) => {
      setProduct((prev) => ({ ...prev, tags: [...prev.tags, tag] }));
    });
  };

  const handleRemoveTag = (tag: string): void => {
    setProduct((prev) => ({
      ...prev,
      tags: prev.tags.filter((t) => t !== tag),
    }));
  };

  const handleIncrease = (): void => {
    setProduct((prev) => ({ ...prev, quantity: prev.quantity + 1 }));
  };

  const handleDecrease = (): void => {
    setProduct((prev) => ({
      ...prev,
      quantity: prev.quantity - 1,
    }));
  };

  return (
   <div>
      <form>
        <input
          type="text"
          name="title"
          onChange={handleChange}
          placeholder="Title"
        />
        <input
          type="text"
          name="desc"
          onChange={handleChange}
          placeholder="Desc"
        />
        <input
          type="number"
          name="price"
          onChange={handleChange}
          placeholder="Price"
        />
        <p>Category:</p>
        <select name="category" id="category" onChange={handleChange}>
          <option value="sneakers">Sneakers</option>
          <option value="tshirts">T-shirts</option>
          <option value="jeans">Jeans</option>
        </select>
        <p>Tags:</p>
        <textarea
          ref={tagRef}
          placeholder="Seperate tags with commas..."
        ></textarea>
        <button type="button" onClick={handleTags}>
          Add Tags
        </button>
        <div className="tags">
          {product.tags.map((tag) => (
            <small key={tag} onClick={() => handleRemoveTag(tag)}>
              {tag}
            </small>
          ))}
        </div>
        <div className="quantity">
          <button type="button" onClick={handleDecrease}>
            -
          </button>
          <span>Quantity ({product.quantity})</span>
          <button type="button" onClick={handleIncrease}>
            +
          </button>
        </div>
      </form>
    </div>
  );
};
```

`useReducer`

```tsx
// formReducer.tsx
interface FormState {
  title: string;
  desc: string;
  price: number;
  category: string;
  tags: string[];
  images: {
    sm: string;
    md: string;
    lg: string;
  };
  quantity: number;
}

interface FormAction {
  type: string;
  payload?: any;
}

export const INITIAL_STATE: FormState = {
  title: "",
  desc: "",
  price: 0,
  category: "",
  tags: [],
  images: {
    sm: "",
    md: "",
    lg: "",
  },
  quantity: 0,
};

export const formReducer = (state: FormState, action: FormAction): FormState => {
  switch (action.type) {
    case "CHANGE_INPUT":
      return {
        ...state,
        [action.payload.name]: action.payload.value,
      };
    case "ADD_TAG":
      return {
        ...state,
        tags: [...state.tags, action.payload],
      };
    case "REMOVE_TAG":
      return {
        ...state,
        tags: state.tags.filter((tag) => tag !== action.payload),
      };
    case "INCREASE":
      return {
        ...state,
        quantity: state.quantity + 1,
      };
    case "DECREASE":
      return { ...state, quantity: state.quantity - 1 };
    default:
      return state;
  }
};

// Form.tsx
import React, { useReducer, useRef, useState } from "react";
import { formReducer, INITIAL_STATE } from "./formReducer";

const Form = () => {
  const [state, dispatch] = useReducer(formReducer, INITIAL_STATE);
  const tagRef = useRef<HTMLTextAreaElement>(null);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLSelectElement>) => {
    dispatch({
      type: "CHANGE_INPUT",
      payload: { name: e.target.name, value: e.target.value },
    });
  };

  const handleTags = () => {
    const tags = tagRef.current?.value.split(",");
    if (tags) {
      tags.forEach((tag) => {
        dispatch({ type: "ADD_TAG", payload: tag });
      });
    }
  };

  return (
    <div>
      <form>
        <input
          type="text"
          placeholder="Title"
          onChange={handleChange}
          name="title"
          value={state.title}
        />
        <input
          type="text"
          placeholder="Desc"
          onChange={handleChange}
          name="desc"
          value={state.desc}
        />
        <input
          type="number"
          placeholder="Price"
          onChange={handleChange}
          name="price"
          value={state.price}
        />
        <p>Category:</p>
        <select onChange={handleChange} name="category" value={state.category}>
          <option value="sneakers">Sneakers</option>
          <option value="tshirts">T-shirts</option>
          <option value="jeans">Jeans</option>
        </select>
        <p>Tags:</p>
        <textarea ref={tagRef} placeholder="Seperate tags with commas..."></textarea>
        <button onClick={handleTags} type="button">
          Add Tags
        </button>
        <div className="tags">
          {state.tags.map((tag) => (
            <small
              onClick={() => dispatch({ type: "REMOVE_TAG", payload: tag })}
              key={tag}
            >
              {tag}
            </small>
          ))}
        </div>
        <div className="quantity">
          <button onClick={() => dispatch({ type: "DECREASE" })} type="button">
            -
          </button>
          <span>Quantity ({state.quantity})</span>
          <button onClick={() => dispatch({ type: "INCREASE" })} type="button">
            +
          </button>
        </div>
      </form>
    </div>
  );
};
```
