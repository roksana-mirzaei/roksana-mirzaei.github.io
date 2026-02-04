---
layout: default
title: Blog
permalink: /blog/
---

<div class="blog-container">
  <h1>Blog</h1>

  <div class="blog-posts">
    {% for post in site.posts %}
      {% unless post.categories contains 'jekyll' and post.categories contains 'update' %}
        <div class="blog-post-item">
          {% if post.image %}
            <img src="{{ post.image | relative_url }}" alt="{{ post.title }}" class="blog-post-image">
          {% endif %}
          <div class="blog-post-content">
            <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
            <p class="blog-post-date">{{ post.date | date: "%B %d, %Y" }}</p>
            <p class="blog-post-excerpt">{{ post.excerpt }}</p>
            <a href="{{ post.url }}" class="read-more">Read more →</a>
          </div>
        </div>
      {% endunless %}
    {% endfor %}
  </div>
</div>

<style>
  .blog-container {
    max-width: 1100px;
    margin: 0 auto;
    padding: 2rem 1rem;
  }

  .blog-container h1 {
    margin-bottom: 2rem;
  }

  .blog-posts {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  .blog-post-item {
    display: flex;
    gap: 1.5rem;
    align-items: flex-start;
    background: var(--bg-primary);
    border: 1px solid var(--border-color);
    border-radius: 4px;
    box-shadow: 0 10px 30px var(--shadow-md);
    overflow: hidden;
    padding: 1.5rem;
    transition: transform 0.2s ease, box-shadow 0.2s ease, border-color 0.2s ease, background-color 0.2s ease;
  }

  .blog-post-item:hover {
    transform: translateY(-4px);
    box-shadow: 0 16px 40px var(--shadow-md);
    border-color: var(--accent-color);
    background: var(--bg-secondary);
  }

  .blog-post-image {
    width: 300px;
    height: 230px;
    object-fit: cover;
    border-radius: 2px;
    flex-shrink: 0;
    background: var(--bg-tertiary);
  }

  .blog-post-content {
    flex: 1;
    display: flex;
    flex-direction: column;
  }

  .blog-post-item h2 {
    margin: 0 0 0.5rem 0;
    font-size: 1.35rem;
    line-height: 1.4;
  }

  .blog-post-item h2 a {
    color: var(--text-primary);
    text-decoration: none;
  }

  .blog-post-item h2 a:hover {
    color: var(--accent-color);
  }

  .blog-post-date {
    color: var(--text-tertiary);
    font-size: 0.95rem;
    margin-bottom: 0.75rem;
  }

  .blog-post-excerpt {
    color: var(--text-secondary);
    line-height: 1.7;
    margin-bottom: 1rem;
    flex-grow: 1;
  }

  .read-more {
    color: var(--accent-color);
    text-decoration: none;
    font-weight: 600;
    display: inline-block;
    margin-top: auto;
  }

  .read-more:hover {
    text-decoration: underline;
  }

  /* Responsive layout for blog cards */
  @media (max-width: 960px) {
    .blog-post-item {
      flex-direction: column;
      align-items: stretch;
    }

    .blog-post-image {
      width: 100%;
      height: 220px;
    }
  }

  @media (max-width: 720px) {
    .blog-container {
      padding: 1.5rem 1rem;
    }

    .blog-post-item {
      padding: 1rem;
      gap: 1rem;
    }

    .blog-post-image {
      height: 190px;
    }

    .blog-post-item h2 {
      font-size: 1.2rem;
    }
  }
</style>
