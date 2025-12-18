+++
title = 'Lessons from Leading a Major Refactor'
date = 2025-12-17T10:41:45+08:00
draft = false
+++

Recently, I led a major refactor on my team, and here are the lessons I learned from it. None of them are groundbreaking—they’re simple principles we’ve all read about countless times. Yet, as this experience reminded me, they’re surprisingly difficult to put into practice.

### Naming Conventions

Perhaps because English isn’t my first language, I pay extra attention to naming functions, parameters, and constants clearly and concisely. A good name should be self-explanatory—when you read it, you immediately understand what it represents.

However, I noticed that within my team, naming conventions often came second. This led to unnecessary ambiguity and made the code harder to reason about over time.

### One Function Does One Thing

In an ideal world, no function should exceed 30 lines of code. As a general rule, smaller functions are easier to understand, test, and maintain, while large functions should be refactored into smaller ones.

That said, both extremes have drawbacks. Long functions are difficult for reviewers to follow, but breaking logic into too many tiny functions can force reviewers to constantly jump around the code. Finding the right balance is key.

My approach is simple: one function should do one thing, and do it well. This strikes a good balance between readability and structure. An additional benefit is that such functions tend to have fewer side effects, making the code safer and easier to reason about.


### No Hidden Logic

This topic overlaps a bit with the previous one, but it’s important enough to call out separately.

Consider a common example: when updating an item in a service, we often need to record metadata such as updatedAt and updatedBy. The best practice is to keep all of this logic in the same place. If the service updates the item, it should also update these related fields directly—instead of relying on listeners, hooks, or other indirect mechanisms.

Hidden logic makes a project difficult to maintain, especially as team members come and go. Over time, these implicit behaviors fade from collective memory. Eventually, someone modifies the service code without realizing the hidden dependencies, and what was once “clever” turns into a P1 incident.

### Keep Controllers Thin

As the system grows, controllers tend to become thick. They slowly accumulate orchestration logic—validation, conditional branching, data transformation, and coordination between multiple services. Over time, this makes controllers hard to read, hard to test, and risky to modify. 

Here’s a simplified example.

In the first version, the controller is responsible not only for handling the request, but also for orchestrating business rules and permission check:

```ts
const item = await this.service.getItem(item)

// validate user permission
if (item.user !== req.user) {
  throw new PermissionError()
}

// more logic...
```

Permission checks and follow-up logic are embedded directly in the controller. As more rules are added, this pattern quickly leads to bloated controllers and duplicated logic across endpoints.

After refactoring, the controller delegates intent instead of implementation:

```ts
const item = await this.service.getItem({
  itemId,
  userId: req.userId,
})

// returns 404 or throws permission error internally
```

The controller now passes all necessary context to the service and focuses solely on request handling. Permission checks and error handling are centralized inside the service, where business rules belong.

Most importantly, the system becomes easier to evolve. When business rules change, there is a single place to update them—without hunting through multiple controllers.