<template>
    <div class="ww-task-manager bg-white p-4 border rounded">
        <!-- Task Title Section -->
        <div class="mb-6">
            <h2 class="text-xl font-bold">{{ content?.data?.task?.title || 'Task Title' }}</h2>
            <button @click="updateTaskField('title', 'Updated Title')" 
                    class="mt-2 bg-blue-500 text-white px-3 py-1 rounded hover:bg-blue-600">
                Update Title
            </button>
        </div>

        <!-- Subtasks Section -->
        <div class="mb-6">
            <h3 class="text-lg font-semibold mb-3">Subtasks</h3>
            <div v-if="content?.data?.task?.subtasks?.length" 
                 class="space-y-4">
                <div v-for="subtask in content.data.task.subtasks" 
                     :key="subtask.id"
                     class="border p-3 rounded">
                    <div class="flex justify-between items-center mb-2">
                        <h4 class="font-medium">{{ subtask.title }}</h4>
                        <button @click="updateSubtask(subtask.id, { title: 'Updated ' + subtask.title })"
                                class="bg-green-500 text-white px-2 py-1 rounded text-sm hover:bg-green-600">
                            Update
                        </button>
                    </div>
                    <div class="space-y-2">
                        <div v-for="(item, index) in subtask.checklist" 
                             :key="index"
                             class="flex items-center">
                            <input type="checkbox" 
                                   :checked="item.completed"
                                   @change="updateChecklistItem(subtask.id, index, !item.completed)"
                                   class="mr-2">
                            <span>{{ item.item }}</span>
                        </div>
                    </div>
                </div>
            </div>
            <div v-else class="text-gray-500 italic">
                No subtasks available
            </div>
        </div>

        <!-- Comments Section -->
        <div class="mb-6">
            <h3 class="text-lg font-semibold mb-3">Comments</h3>
            <div v-if="content?.data?.task?.comments?.length" 
                 class="space-y-3">
                <div v-for="comment in content.data.task.comments" 
                     :key="comment.id"
                     class="bg-gray-50 p-3 rounded">
                    <div class="flex justify-between text-sm text-gray-600 mb-1">
                        <span class="font-medium">{{ comment.user.name }}</span>
                        <span>{{ formatDate(comment.timestamp) }}</span>
                    </div>
                    <p>{{ comment.content }}</p>
                </div>
            </div>
            <div v-else class="text-gray-500 italic mb-3">
                No comments yet
            </div>
            <button @click="addComment" 
                    class="bg-purple-500 text-white px-3 py-1 rounded hover:bg-purple-600">
                Add Comment
            </button>
        </div>

        <!-- Debug Section -->
        <div class="mt-6 p-3 bg-gray-100 rounded">
            <h4 class="font-semibold mb-2">Current Data:</h4>
            <pre class="text-xs overflow-auto">{{ JSON.stringify(content, null, 2) }}</pre>
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
                        id: "task-001",
                        title: "Default Task",
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
                if (this.content?.data?.task?.id) {
                    await wwLib.wwData.update('tasks', this.content.data.task.id, {
                        [field]: value
                    });
                }
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
            if (!this.content?.data?.task?.subtasks) return;
            
            const subtasks = [...this.content.data.task.subtasks];
            const index = subtasks.findIndex(st => st.id === subtaskId);
            
            if (index !== -1) {
                subtasks[index] = { ...subtasks[index], ...updates };
                try {
                    if (this.content?.data?.task?.id) {
                        await wwLib.wwData.update('tasks', this.content.data.task.id, {
                            subtasks: subtasks
                        });
                    }
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

        async updateChecklistItem(subtaskId, itemIndex, completed) {
            if (!this.content?.data?.task?.subtasks) return;

            const subtasks = JSON.parse(JSON.stringify(this.content.data.task.subtasks));
            const subtask = subtasks.find(st => st.id === subtaskId);
            
            if (subtask?.checklist?.[itemIndex] !== undefined) {
                subtask.checklist[itemIndex].completed = completed;
                try {
                    if (this.content?.data?.task?.id) {
                        await wwLib.wwData.update('tasks', this.content.data.task.id, {
                            subtasks: subtasks
                        });
                    }
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
                    console.error('Failed to update checklist item:', err);
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

            const comments = [...(this.content?.data?.task?.comments || []), newComment];

            try {
                if (this.content?.data?.task?.id) {
                    await wwLib.wwData.update('tasks', this.content.data.task.id, {
                        comments: comments
                    });
                }
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
            if (!timestamp) return '';
            return new Date(timestamp).toLocaleString();
        }
    },

    mounted() {
        console.log('Component mounted with content:', this.content);
    }
}
</script>

<style scoped>
.ww-task-manager {
    min-height: 100px;
    width: 100%;
}
</style>