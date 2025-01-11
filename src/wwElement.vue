<template>
    <div class="ww-task-manager">
        <div class="task-content">
            <!-- Task Title -->
            <div class="task-header">
                <h2 class="task-title">{{ content.data.task.title }}</h2>
                <div class="task-actions">
                    <button @click="updateTaskField('title', 'Updated Title')" class="action-button">
                        Update Title
                    </button>
                </div>
            </div>

            <!-- Subtasks -->
            <div class="subtasks-section">
                <h3>Subtasks</h3>
                <div v-for="subtask in content.data.task.subtasks" 
                     :key="subtask.id" 
                     class="subtask-item">
                    <div class="subtask-header">
                        <h4>{{ subtask.title }}</h4>
                        <button @click="updateSubtask(subtask.id, { title: 'Updated ' + subtask.title })">
                            Update
                        </button>
                    </div>
                    <div class="checklist">
                        <div v-for="(item, index) in subtask.checklist" 
                             :key="index"
                             class="checklist-item">
                            <input type="checkbox" 
                                   :checked="item.completed"
                                   @change="updateChecklistItem(subtask.id, index, !item.completed)">
                            <span>{{ item.item }}</span>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Comments -->
            <div class="comments-section">
                <h3>Comments</h3>
                <div v-for="comment in content.data.task.comments" 
                     :key="comment.id" 
                     class="comment">
                    <div class="comment-header">
                        <span class="comment-author">{{ comment.user.name }}</span>
                        <span class="comment-time">{{ formatDate(comment.timestamp) }}</span>
                    </div>
                    <p class="comment-content">{{ comment.content }}</p>
                </div>
                <button @click="addComment" class="add-comment">
                    Add Comment
                </button>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "Task Manager",
    props: {
        content: {
            type: Object,
            default: () => ({
                data: {
                    task: {
                        id: "",
                        title: "New Task",
                        subtasks: [],
                        comments: []
                    }
                }
            })
        }
    },

    methods: {
        async updateTaskField(field, value) {
            try {
                await wwLib.wwData.update('tasks', this.content.data.task.id, {
                    [field]: value
                });
                this.$emit('update:content', {
                    ...this.content,
                    data: {
                        ...this.content.data,
                        task: {
                            ...this.content.data.task,
                            [field]: value
                        }
                    }
                });
            } catch (err) {
                console.error('Failed to update task:', err);
            }
        },

        async updateSubtask(subtaskId, updates) {
            const subtasks = [...this.content.data.task.subtasks];
            const index = subtasks.findIndex(st => st.id === subtaskId);
            
            if (index !== -1) {
                subtasks[index] = { ...subtasks[index], ...updates };
                try {
                    await wwLib.wwData.update('tasks', this.content.data.task.id, {
                        subtasks: subtasks
                    });
                    this.$emit('update:content', {
                        ...this.content,
                        data: {
                            ...this.content.data,
                            task: {
                                ...this.content.data.task,
                                subtasks
                            }
                        }
                    });
                } catch (err) {
                    console.error('Failed to update subtask:', err);
                }
            }
        },

        async addComment() {
            const newComment = {
                id: `comment-${Date.now()}`,
                user: { id: 'current-user', name: 'Current User' },
                content: 'New comment',
                timestamp: new Date().toISOString()
            };

            const comments = [...(this.content.data.task.comments || []), newComment];

            try {
                await wwLib.wwData.update('tasks', this.content.data.task.id, {
                    comments: comments
                });
                this.$emit('update:content', {
                    ...this.content,
                    data: {
                        ...this.content.data,
                        task: {
                            ...this.content.data.task,
                            comments
                        }
                    }
                });
            } catch (err) {
                console.error('Failed to add comment:', err);
            }
        },

        formatDate(timestamp) {
            return new Date(timestamp).toLocaleString();
        }
    },

    mounted() {
        // Subscribe to real-time updates if needed
        if (this.content.data.task.id) {
            wwLib.wwData.subscribe('tasks', this.content.data.task.id, (newData) => {
                this.$emit('update:content', {
                    ...this.content,
                    data: newData
                });
            });
        }
    },

    beforeDestroy() {
        // Clean up subscription
        if (this.content.data.task.id) {
            wwLib.wwData.unsubscribe('tasks', this.content.data.task.id);
        }
    }
}
</script>

<style scoped>
.ww-task-manager {
    padding: 1rem;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.task-header {
    margin-bottom: 1.5rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.subtask-item {
    margin: 1rem 0;
    padding: 1rem;
    border: 1px solid #e5e7eb;
    border-radius: 0.375rem;
}

.comment {
    margin: 0.75rem 0;
    padding: 0.75rem;
    background-color: #f9fafb;
    border-radius: 0.375rem;
}

button {
    padding: 0.5rem 1rem;
    background-color: #3b82f6;
    color: white;
    border-radius: 0.375rem;
    border: none;
    cursor: pointer;
}

button:hover {
    background-color: #2563eb;
}
</style>