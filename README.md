 # Task Manager Component for Weweb

A component for managing hierarchical tasks with subtasks, comments, and attachments.

## Features
- Manage main tasks and subtasks
- Track completion status
- Add and manage comments
- Handle file attachments
- Real-time updates
- Progress tracking

## Installation
1. In Weweb editor, go to Components > Add Component
2. Add this repository URL
3. Configure your component settings

## Structure
The component uses the following data structure:
```json
{
  "task": {
    "id": "string",
    "title": "string",
    "description": "string",
    "status": "string",
    "progress": "string",
    "subtasks": [
      {
        "id": "string",
        "title": "string",
        "completed": "boolean"
      }
    ],
    "comments": [
      {
        "id": "string",
        "content": "string",
        "userId": "string",
        "timestamp": "string"
      }
    ]
  }
}
```

## Usage
1. Add the component to your Weweb page
2. Configure the data collection in Weweb's Data tab
3. Bind the component to your task data
4. Use the provided UI to manage tasks

## Development
To modify this component:
1. Clone the repository
2. Make your changes
3. Test in Weweb
4. Submit a pull request

## License
MIT
