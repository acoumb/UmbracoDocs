---
description: >-
    Variable interpolation syntax for prompt templates.
---

# Template Syntax

Prompt templates support variable interpolation using double curly braces: `{{variable}}`.

## Basic Variables

Variables in templates are resolved from the runtime context. When executing a prompt from the backoffice (via property actions), the context is automatically populated with entity data and property values.

```
Translate the following text to {{language}}:

{{text}}
```

Variables can reference:

- Entity properties (name, content type, etc.)
- Content property values by alias
- Custom context items passed in the request

## Nested Paths

Use dot notation to access nested object properties:

```
User: {{user.name}}
Email: {{user.email}}
Company: {{user.company.name}}
```

## Dictionary and Array Access

Access dictionary values and array elements using bracket notation:

```
First item: {{items[0]}}
Specific key: {{data["my-key"]}}
Combined: {{users[0].name}}
```

Supported path patterns:

- Simple property: `{{name}}`
- Nested property: `{{user.name}}`
- Array index: `{{items[0]}}`
- Dictionary key: `{{data["key"]}}` or `{{data['key']}}`
- Combined: `{{users[0].company.name}}`

{% hint style="info" %}
Path resolution is case-insensitive by default.
{% endhint %}

## Image Variables

Use the `image:` prefix to include images from content properties:

```
Describe this image:
{{image:heroImage}}

Write alt text for:
{{image:umbracoFile}}
```

The image processor:

1. Fetches the entity from Umbraco
2. Gets the media property value
3. Resolves the image content
4. Includes the image data in the AI request

{% hint style="info" %}
Image variables require the entity to have a media picker or upload property with the specified alias.
{% endhint %}

## Executing Prompts

### From Property Actions

When prompts execute from property actions in the backoffice, the context is automatically populated:

- Entity ID and type
- Property alias being edited
- Culture and segment (if applicable)
- All property values from the entity

Templates can reference this context directly:

```
You are editing "{{name}}" ({{contentType}}).

Current title: {{pageTitle}}
Current summary: {{summary}}

Suggest improvements for SEO.
```

### From Code

Execute prompts programmatically using `IAIPromptService`:

{% code title="ExecutePromptWithContext.cs" %}

```csharp
var result = await _promptService.ExecutePromptAsync(
    promptId,
    new AIPromptExecutionRequest
    {
        EntityId = contentId.ToString(),
        EntityType = "document",
        EntityContext = content.GetValue<string>("bodyText"),
        Variables = new Dictionary<string, string>
        {
            ["language"] = "French",
            ["culture"] = "en-US"
        }
    });
```

{% endcode %}

The `AIPromptExecutionRequest` properties:

| Property        | Type                          | Description                           |
| --------------- | ----------------------------- | ------------------------------------- |
| `EntityId`      | `string?`                     | The content or media item ID          |
| `EntityType`    | `string?`                     | "document" or "media"                 |
| `EntityContext`  | `string?`                     | Contextual data about the entity      |
| `Variables`     | `IDictionary<string, string>` | Variables to resolve in the template  |

## Variables

Variables provide additional information to the prompt template. Pass a dictionary of key-value pairs that map to `{{variable}}` placeholders:

{% code title="Variables Example" %}

```csharp
var result = await _promptService.ExecutePromptAsync(
    promptId,
    new AIPromptExecutionRequest
    {
        Variables = new Dictionary<string, string>
        {
            ["language"] = "French",
            ["tone"] = "professional"
        }
    });
```

{% endcode %}

Each key in the `Variables` dictionary can be referenced in the template using the `{{key}}` syntax.

## Combining Variables

Mix different variable types in a single template:

```
## Content: {{name}}

Current text:
{{bodyText}}

{{image:featuredImage}}

Generate a social media post about this content.
Target language: {{targetLanguage}}
```

## Best Practices

### Be Explicit

Instead of:

```
Write about {{topic}}
```

Use:

```
Write a 500-word blog post about {{topic}}.
Include an introduction, 3 main points, and a conclusion.
Use a conversational tone.
```

### Document Variables

Add comments to complex templates:

```
<!--
Required variables:
- topic: Main subject
- audience: Target readers
- tone: Writing style
-->

Write about {{topic}} for {{audience}}.
Use a {{tone}} tone.
```

### Structure Complex Templates

For complex prompts, use clear sections:

```
## Context
You are a {{role}} assistant.

## Task
{{taskDescription}}

## Input
{{inputContent}}

## Requirements
Respond in {{language}}.
Keep the response under {{maxWords}} words.
```

## Related

- [Concepts](concepts.md) - Prompt template concepts
- [Scoping](scoping.md) - Allow and deny rules
- [Property Actions](property-actions.md) - Using prompts in the backoffice
