# 💻 Machine Coding & UI Components Interview Guide

A collection of the most frequently tested Machine Coding components and coding challenges, complete with production-ready implementations and interview rubrics.

---

## 📑 Table of Contents
1. [Machine Coding Evaluation Rubric](#1-machine-coding-evaluation-rubric)
2. [Component 1: Star Rating](#component-1-star-rating)
3. [Component 2: Accessible Modal Dialog with Focus Trap](#component-2-accessible-modal-dialog-with-focus-trap)
4. [Component 3: Autocomplete / Typeahead Search](#component-3-autocomplete--typeahead-search)
5. [Component 4: Accordion (Single & Multi-Expand)](#component-4-accordion-single--multi-expand)
6. [Component 5: Toast Notification System](#component-5-toast-notification-system)
7. [Component 6: Nested Comments System (Recursive Tree)](#component-6-nested-comments-system-recursive-tree)
8. [Component 7: Infinite Scroll with IntersectionObserver](#component-7-infinite-scroll-with-intersectionobserver)
9. [Component 8: Image Carousel / Slider](#component-8-image-carousel--slider)

---

## 1. Machine Coding Evaluation Rubric

In 45-60 minute machine coding rounds, interviewers evaluate you on 5 core pillars:
1. **Working Functionality (40%)**: Core requirements met, edge cases handled.
2. **Code Modularity & Clean Architecture (20%)**: Separation of concerns, custom hooks, reusable components.
3. **Accessibility (a11y) & Semantic HTML (15%)**: Keyboard navigation, correct ARIA roles, focus management.
4. **Performance & Optimization (15%)**: Avoiding unnecessary re-renders, debouncing, clean memory unmounting.
5. **Styling & User Experience (10%)**: Clean, responsive layout with visual feedback (hover, focus, disabled states).

---

## Component 1: Star Rating

### Requirements:
- Hovering previews the rating.
- Clicking sets the rating.
- Mouse leave restores the selected rating.
- Keyboard accessible (left/right arrows).

```jsx
import React, { useState } from 'react';

export function StarRating({ totalStars = 5, initialRating = 0, onChange }) {
  const [rating, setRating] = useState(initialRating);
  const [hoverRating, setHoverRating] = useState(0);

  const handleClick = (value) => {
    setRating(value);
    if (onChange) onChange(value);
  };

  return (
    <div
      role="radiogroup"
      aria-label="Star Rating"
      style={{ display: 'flex', gap: '8px', cursor: 'pointer' }}
      onMouseLeave={() => setHoverRating(0)}
    >
      {Array.from({ length: totalStars }, (_, i) => {
        const starValue = i + 1;
        const isFilled = starValue <= (hoverRating || rating);

        return (
          <button
            key={starValue}
            type="button"
            role="radio"
            aria-checked={rating === starValue}
            aria-label={`${starValue} of ${totalStars} stars`}
            style={{
              background: 'none',
              border: 'none',
              fontSize: '2rem',
              color: isFilled ? '#fbbf24' : '#d1d5db',
              cursor: 'pointer',
              padding: 0
            }}
            onClick={() => handleClick(starValue)}
            onMouseEnter={() => setHoverRating(starValue)}
          >
            ★
          </button>
        );
      })}
    </div>
  );
}
```

---

## Component 2: Accessible Modal Dialog with Focus Trap

### Requirements:
- Backdrop overlay click closes modal.
- `Escape` key closes modal.
- Traps focus inside the modal when open (`Tab` cycles through modal elements).
- Restores focus to the triggering element when closed.

```jsx
import React, { useEffect, useRef } from 'react';

export function Modal({ isOpen, onClose, title, children }) {
  const modalRef = useRef(null);
  const previousActiveElement = useRef(null);

  useEffect(() => {
    if (!isOpen) return;

    // Save previous active element to restore focus on close
    previousActiveElement.current = document.activeElement;

    const handleKeyDown = (e) => {
      if (e.key === 'Escape') {
        onClose();
        return;
      }

      if (e.key === 'Tab') {
        const focusableElements = modalRef.current.querySelectorAll(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );
        const firstEl = focusableElements[0];
        const lastEl = focusableElements[focusableElements.length - 1];

        if (e.shiftKey && document.activeElement === firstEl) {
          e.preventDefault();
          lastEl.focus();
        } else if (!e.shiftKey && document.activeElement === lastEl) {
          e.preventDefault();
          firstEl.focus();
        }
      }
    };

    document.addEventListener('keydown', handleKeyDown);

    // Focus the first element inside the modal
    const focusable = modalRef.current?.querySelector('button, [href], input');
    if (focusable) focusable.focus();

    return () => {
      document.removeEventListener('keydown', handleKeyDown);
      if (previousActiveElement.current) {
        previousActiveElement.current.focus();
      }
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div
      role="presentation"
      onClick={onClose}
      style={{
        position: 'fixed',
        inset: 0,
        backgroundColor: 'rgba(0, 0, 0, 0.5)',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        zIndex: 1000
      }}
    >
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        onClick={(e) => e.stopPropagation()} // Prevent closing when clicking content
        style={{
          background: 'white',
          borderRadius: '8px',
          padding: '24px',
          maxWidth: '500px',
          width: '90%'
        }}
      >
        <h2 id="modal-title">{title}</h2>
        <div>{children}</div>
        <button onClick={onClose} style={{ marginTop: '16px' }}>Close</button>
      </div>
    </div>
  );
}
```

---

## Component 3: Autocomplete / Typeahead Search

```jsx
import React, { useState, useEffect, useRef } from 'react';

export function Autocomplete({ fetchSuggestions, onSelect }) {
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState([]);
  const [isOpen, setIsOpen] = useState(false);
  const [selectedIndex, setSelectedIndex] = useState(-1);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!query.trim()) {
      setSuggestions([]);
      return;
    }

    const controller = new AbortController();
    setLoading(true);

    const timer = setTimeout(async () => {
      try {
        const results = await fetchSuggestions(query, controller.signal);
        setSuggestions(results);
        setIsOpen(true);
      } catch (err) {
        if (err.name !== 'AbortError') console.error(err);
      } finally {
        setLoading(false);
      }
    }, 300);

    return () => {
      clearTimeout(timer);
      controller.abort();
    };
  }, [query, fetchSuggestions]);

  const handleKeyDown = (e) => {
    if (e.key === 'ArrowDown') {
      e.preventDefault();
      setSelectedIndex(prev => (prev < suggestions.length - 1 ? prev + 1 : 0));
    } else if (e.key === 'ArrowUp') {
      e.preventDefault();
      setSelectedIndex(prev => (prev > 0 ? prev - 1 : suggestions.length - 1));
    } else if (e.key === 'Enter' && selectedIndex >= 0) {
      e.preventDefault();
      handleSelect(suggestions[selectedIndex]);
    } else if (e.key === 'Escape') {
      setIsOpen(false);
    }
  };

  const handleSelect = (item) => {
    setQuery(item);
    setIsOpen(false);
    setSelectedIndex(-1);
    if (onSelect) onSelect(item);
  };

  return (
    <div style={{ position: 'relative', width: '300px' }}>
      <input
        type="text"
        role="combobox"
        aria-autocomplete="list"
        aria-expanded={isOpen}
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        onKeyDown={handleKeyDown}
        placeholder="Search..."
        style={{ width: '100%', padding: '8px' }}
      />
      {loading && <span>Loading...</span>}
      {isOpen && suggestions.length > 0 && (
        <ul
          role="listbox"
          style={{
            position: 'absolute',
            top: '100%',
            left: 0,
            right: 0,
            border: '1px solid #ccc',
            background: 'white',
            listStyle: 'none',
            margin: 0,
            padding: 0
          }}
        >
          {suggestions.map((item, idx) => (
            <li
              key={item}
              role="option"
              aria-selected={idx === selectedIndex}
              onClick={() => handleSelect(item)}
              style={{
                padding: '8px',
                cursor: 'pointer',
                backgroundColor: idx === selectedIndex ? '#e5e7eb' : 'transparent'
              }}
            >
              {item}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

## Component 4: Toast Notification System

```jsx
import React, { createContext, useContext, useState, useCallback } from 'react';

const ToastContext = createContext(null);

export function ToastProvider({ children }) {
  const [toasts, setToasts] = useState([]);

  const addToast = useCallback((message, type = 'info', duration = 3000) => {
    const id = Date.now() + Math.random();
    setToasts((prev) => [...prev, { id, message, type }]);

    setTimeout(() => {
      setToasts((prev) => prev.filter((t) => t.id !== id));
    }, duration);
  }, []);

  const removeToast = useCallback((id) => {
    setToasts((prev) => prev.filter((t) => t.id !== id));
  }, []);

  return (
    <ToastContext.Provider value={{ addToast }}>
      {children}
      <div
        style={{
          position: 'fixed',
          bottom: '20px',
          right: '20px',
          display: 'flex',
          flexDirection: 'column',
          gap: '10px',
          zIndex: 9999
        }}
      >
        {toasts.map((toast) => (
          <div
            key={toast.id}
            role="alert"
            style={{
              padding: '12px 20px',
              borderRadius: '6px',
              color: 'white',
              backgroundColor: toast.type === 'error' ? '#ef4444' : '#10b981',
              boxShadow: '0 4px 6px rgba(0,0,0,0.1)',
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'space-between',
              minWidth: '240px'
            }}
          >
            <span>{toast.message}</span>
            <button
              onClick={() => removeToast(toast.id)}
              style={{ background: 'none', border: 'none', color: 'white', cursor: 'pointer', marginLeft: '12px' }}
            >
              ✕
            </button>
          </div>
        ))}
      </div>
    </ToastContext.Provider>
  );
}

export const useToast = () => useContext(ToastContext);
```

---

## Component 5: Nested Comments System (Recursive Tree)

```jsx
import React, { useState } from 'react';

export function CommentItem({ comment, onAddReply }) {
  const [isReplying, setIsReplying] = useState(false);
  const [replyText, setReplyText] = useState('');

  const handleReplySubmit = () => {
    if (!replyText.trim()) return;
    onAddReply(comment.id, replyText);
    setReplyText('');
    setIsReplying(false);
  };

  return (
    <div style={{ marginLeft: '20px', marginTop: '12px', borderLeft: '2px solid #e5e7eb', paddingLeft: '12px' }}>
      <p style={{ margin: 0, fontWeight: 'bold' }}>{comment.author}</p>
      <p style={{ margin: '4px 0' }}>{comment.text}</p>
      <button onClick={() => setIsReplying(!isReplying)}>
        {isReplying ? 'Cancel' : 'Reply'}
      </button>

      {isReplying && (
        <div style={{ marginTop: '8px' }}>
          <input
            type="text"
            value={replyText}
            onChange={(e) => setReplyText(e.target.value)}
            placeholder="Write a reply..."
          />
          <button onClick={handleReplySubmit}>Submit</button>
        </div>
      )}

      {comment.replies && comment.replies.length > 0 && (
        <div>
          {comment.replies.map((reply) => (
            <CommentItem key={reply.id} comment={reply} onAddReply={onAddReply} />
          ))}
        </div>
      )}
    </div>
  );
}
```
