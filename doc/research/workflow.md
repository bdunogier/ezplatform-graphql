## GraphQL workflow support

## MVP Use-case: review queue

### Abstract

Focus on the review queue use-case: reviewers use an app to get notified of new items that require their attention, view the items, write comments, and advance them to the next steps.

It matches the admin's Review queue dashboard block.

### Features

- get workflow items that awaits my attention, including the content item
- get access to the discussion (messages ?)
- progress the item through the workflow (mutation)
- get notifications. Ideally with subscriptions support

### Schema

#### Viewer field

The current user's workflow items should be accessed through the `viewer` field:
```
{
  viewer {
    reviewQueue {
    }
  }
}
```

What should that field be named exactly ?

- `workflow` is quite plain, and straight to the point. But what does it yield ?
- `workflowItems` clearly indicates that we get items, but do we want to get more data about workflows at all ?
- `reviewQueue` matches the dashboard's terminology. It yields a connection to `ReviewQueueItem` objects for 



#### The review queue: `viewer { reviewQueue }`

Yields the items that are up for review for the current user, as a connection of `ReviewQueueItem ` objects:

```
{
    viewer {
        reviewQueue {
            edges {
                node {
                    content { _name _url }
                    stage { label color }
                    workflow { name }
                }
            }
        }
    }
}
```

##### Type: `ReviewQueueItem`

An workflow item for review. Maps to a `WorkflowMetadata` instance.

```
type ReviewQueueItem {
    id: ID!
    name: String    
    workflow: Workflow
    stage: WorkflowStage
    modifiedLanguage: RepositoryLanguage
    contentItem: DomainContent
    initialOwner: User
    startDate: DateTime
    transitions: "[WorkflowTransition]"
    availableTransitions: "[WorkflowTransition]"
}
```

`availableTransitions`is obtained using:

- ```
  $this->workflowRegistry->getSupportedWorkflows($content);
  foreach ($workflows as $workflow) {
    $workflow->getMarking($content);
    $workflow->getEnabledTransitions($content);
  }
  ```

#### `WorkflowStage` type

```
type WorkflowStage {
    label: String
    color: String
    isLast: Boolean
}
```

##### `Workflow` type

Maps to a `WorkflowDefinitionMetadata` instance.

```
type Workflow {
    name: String
    stages: [WorkflowStage]
    matchers [WorkflowMatcher]
    transitions: [WorkflowTransition]
}
```

#### Filtering the review queue

```
reviewQueue(workflow: <generatedWorkflowIdentifier>)
reviewQueue(contentId: <contentId>)
```

#### Progressing a workflow item

```
mutation ProgressWorkflow {
    applyWorkflowTransition(<contentId>, <workflowId>, "<transition>", "<message>") {
        
    }
}
```

Q: should this be generated from the workflow config ? We could generate `WorkflowTransition` types, based on the workflow (`article`) and transition name (`to_design`), that are used for input to `applyWorkflowTransition`:

```
applyWorkflowTransition()
```

Or with a mutation per workflow type ?

```
applyReviewWorkflowTransition(<contentId>, to_design, "For design")
```

`applyWorkflowTransition` expects a `ReviewWorkflowTransition` as the 2nd parameter. It is an enum generated from the workflow config, with each transition as a value.

#### Schema: build workflows per identifier

>  Rating: 2/5

Add workflows to the schema based on their identifier: `editorial_review` => `editorialReview`.

```
{
  viewer {
    workflows {
      editorialReview {
        stage
        content {
          _name
          _info { id }
        }
      }
    }
  }
}
```
