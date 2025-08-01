# Post Scorer

A Go package that uses OpenAI's GPT to score Reddit posts based on some arbitrary criteria specified in a custom prompt.

## Overview

The scorer evaluates Reddit posts and returns a slice of `ScoredPost` structs containing:

- The original post
- A relevance score (0-100)
- A reason for the score

## Installation

```bash
go get github.com/JohnPlummer/post-scorer@v0.9.0
```

## Usage

```go
package main

import (
    "context"
    "github.com/JohnPlummer/post-scorer/scorer"
    "github.com/JohnPlummer/reddit-client/reddit"
)

func main() {
    // Initialize the scorer
    s, err := scorer.New(scorer.Config{
        OpenAIKey: "your-api-key",
    })
    if err != nil {
        panic(err)
    }

    // Score some posts
    posts := []*reddit.Post{
        {
            ID:    "post1",
            Title: "Best restaurants in town?",
        },
    }

    scored, err := s.ScorePosts(context.Background(), posts)
    if err != nil {
        panic(err)
    }

    // Use the scored posts
    for _, post := range scored {
        fmt.Printf("Post: %s\nScore: %d\nReason: %s\n\n", 
            post.Post.Title, 
            post.Score, 
            post.Reason)
    }
}
```

## Configuration

The `Config` struct accepts:

- `OpenAIKey` (required): Your OpenAI API key
- `Model` (optional): OpenAI model to use (defaults to GPT-4o-mini)
- `PromptText` (optional): Custom prompt template
- `MaxConcurrent` (optional): For rate limiting

## Advanced Usage

### Per-Request Model Selection

Override the model for specific scoring requests:

```go
// Use GPT-4 for more accurate scoring
scoredPosts, err := scorer.ScorePostsWithOptions(ctx, posts,
    scorer.WithModel("gpt-4"))

// Use GPT-3.5-turbo for faster, cheaper scoring
scoredPosts, err := scorer.ScorePostsWithOptions(ctx, posts,
    scorer.WithModel("gpt-3.5-turbo"))
```

### Custom Prompt Templates

Use Go template syntax for dynamic prompts:

```go
// Template with extra context
template := "Score posts for {{.City}}: {{.Posts}}"
scoredPosts, err := scorer.ScorePostsWithOptions(ctx, posts,
    scorer.WithPromptTemplate(template),
    scorer.WithExtraContext(map[string]string{"City": "Brighton"}))
```

### Scoring with Context

Score posts with additional context data like comments:

```go
contexts := []scorer.ScoringContext{
    {
        Post: post,
        ExtraData: map[string]string{
            "Comments": "Great coffee! Been there many times.",
            "Metadata": "Posted in r/Brighton",
        },
    },
}

// Use a template that includes the extra context
template := `Score this post:
Title: {{range .Contexts}}{{.PostTitle}}{{end}}
Body: {{range .Contexts}}{{.PostBody}}{{end}}
Comments: {{range .Contexts}}{{.Comments}}{{end}}`

scoredPosts, err := scorer.ScorePostsWithContext(ctx, contexts,
    scorer.WithPromptTemplate(template),
    scorer.WithModel("gpt-4o"))
```

## Custom Prompts

Your prompt must instruct the LLM to return JSON in this exact format:

```json
{
  "version": "1.0",
  "scores": [
    {
      "post_id": "<id>",
      "title": "<title>",
      "score": <0-100>,
      "reason": "<explanation>"
    }
  ]
}
```

Critical requirements:

1. Output must be ONLY valid JSON (no markdown or other formatting)
2. All fields are required
3. Score must be between 0-100
4. Every post must receive a score and reason
5. Include `%s` as placeholder for simple prompts, or use Go template syntax for advanced prompts

See `examples/basic/custom_prompt.txt` for a complete example prompt.

## Documentation

For comprehensive documentation, see the `docs/` directory:

- **[Project Overview](docs/project-overview.md)** - Architecture, features, and use cases
- **[Development Setup](docs/development-setup.md)** - Installation, dependencies, and coding standards  
- **[Package Usage](docs/package-usage.md)** - Complete API reference and examples
- **[Key Components](docs/key-components.md)** - Core interfaces and implementation details
- **[Deployment Guide](docs/deployment-guide.md)** - Production deployment and configuration
- **[Troubleshooting](docs/troubleshooting.md)** - Common issues and solutions
- **[Recent Changes](docs/recent-changes.md)** - Latest updates and improvements

## Examples

Check the `examples` directory for complete usage examples, including CSV data loading and custom prompt configuration.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
