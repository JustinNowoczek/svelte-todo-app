<script lang="ts">
	type ClearCompletedType = () => void
	type CompletedTodosType = number

	export let clearCompleted: ClearCompletedType
	export let completedTodos: CompletedTodosType
</script>

<button class:hidden={completedTodos === 0} on:click={clearCompleted} class="clear-completed"
	>Clear completed</button
>
