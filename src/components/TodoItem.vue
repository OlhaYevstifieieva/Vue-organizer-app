<script>
export default {
	name: 'TodoItem',
	props: {
		todo: Object,
		index: Number,
	},
	emits: ['update', 'delete'],
	data() {
		return {
			editing: false,
			newTitle:this.todo.title,
		}
	},
	methods: {
		toggle() {
			this.$emit('update', {
				...this.todo,
				completed: !this.todo.completed
			})
		},
		remove() {
			this.$emit('delete');
		},
		edit() {
			this.editing = true;
			this.newTitle = this.todo.title;
			this.$nextTick(( ) => {
				this.$refs['title-field'].focus();
			});
		},
		rename() {
			if(!this.editing) {
				return;
			}
			this.editing = false;
			if(this.newTitle === this.todo.title) {
				return;
			}
			if(this.newTitle === '') {
				this.remove();
				return;
			}
			this.$emit('update', {
				...this.todo,
				title: this.newTitle
			})
			this.newTitle = '';
		},
	}
}
</script>

<template>
          <div
          class="todo"
           :class="{ completed: todo.completed}"
          >
            <label class="todo__status-label">
              <input
                type="checkbox"
                class="todo__status"
                :checked="todo.completed"
				@change="toggle"
              />
            </label>

            <form v-if="editing" @submit.prevent="rename">
              <input
                type="text"
                class="todo__title-field"
                placeholder="Empty todo will be deleted"
                v-model.trim="newTitle"
				@keyup.esc="editing=false"
				ref="title-field"
				@blur="rename"
              />
            </form>
            <template v-else>
              <span class="todo__title" @dblclick="edit">{{ todo.title }}</span>
            <button type="button" class="todo__remove" v-on:click="remove">×</button>
            </template>

            <div class="modal overlay" :class="{ 'is-active': false}">
              <div class="modal-background has-background-white-ter">
              </div>
              <div class="loader" >
              </div>
            </div>
          </div>
</template>

<style>
</style>
