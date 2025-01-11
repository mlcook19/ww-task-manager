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
