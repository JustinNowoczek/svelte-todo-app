<script lang="ts">
	import Todo from '$root/components/Todo.svelte'
	import type { ITodo, FiltersType } from '$root/types/todo'
	import AddTodo from './AddTodo.svelte'
	import TodosLeft from './TodosLeft.svelte'
	import FilteredTodos from './FilteredTodos.svelte'
	import ClearTodo from './ClearTodo.svelte'
	import { useStorage } from '$root/stores/useStorage'
	import { tick } from 'svelte'

	let todos = useStorage<ITodo[]>('$todos', [])

	let selectedFilter: FiltersType = 'all'
	let filtering = false

	$: todosAmount = $todos.length
	$: incompleteTodos = $todos.reduce((total, { completed }) => (completed ? total + 1 : total), 0)
	$: filteredTodos = filterTodo($todos, selectedFilter)
	$: completedTodos = $todos.filter((todo) => todo.completed).length
	$: duration = filtering ? 0 : 250

	function generateRandomId(): string {
		return Math.random().toString(16).slice(2)
	}

	function addTodo(todo: string): void {
		let newTodo: ITodo = {
			id: generateRandomId(),
			text: todo,
			completed: false,
		}

		$todos = [...$todos, newTodo]
	}

	function toggleCompleted(e: MouseEvent): void {
		let { checked } = e.target as HTMLInputElement

		$todos = $todos.map((todo) => ({
			...todo,
			checked,
		}))
	}

	function completeTodo(id: string): void {
		$todos = $todos.map((todo) => (todo.id !== id ? todo : { ...todo, completed: !todo.completed }))
	}

	function removeTodo(id: string): void {
		$todos = $todos.filter((todo) => todo.id !== id)
	}

	function editTodo(id: string, newTodo: string): void {
		$todos = $todos.map((todo) => (todo.id !== id ? todo : { ...todo, text: newTodo }))
	}

	async function setFilter(newFilter: FiltersType): Promise<void> {
		filtering = true
		await tick()
		selectedFilter = newFilter
		await tick()
		filtering = false
	}

	function filterTodo($todos: ITodo[], filter: FiltersType): ITodo[] {
		return $todos.filter(
			({ completed }) =>
				filter === 'all' ||
				(filter === 'active' && !completed) ||
				(filter === 'completed' && completed)
		)
	}

	function clearCompleted(): void {
		$todos = $todos.filter((todo) => todo.completed !== true)
	}
</script>

<main>
	<h1 class="title">Todos</h1>

	<section class="todos">
		<AddTodo {addTodo} {toggleCompleted} {todosAmount} />

		{#if todosAmount}
			<ul class="todo-list">
				{#each filteredTodos as todo}
					<Todo {todo} {completeTodo} {removeTodo} {editTodo} {duration} />
				{/each}
			</ul>

			<div class="actions">
				<TodosLeft {incompleteTodos} />
				<FilteredTodos {selectedFilter} {setFilter} />
				<ClearTodo {clearCompleted} {completedTodos} />
			</div>
		{/if}
	</section>
</main>

<style>
	/* Todos */

	.title {
		font-size: var(--font-80);
		font-weight: inherit;
		text-align: center;
		color: var(--color-title);
	}

	.todos {
		--width: 500px;
		--todos-bg: hsl(0 0% 98%);
		--todos-text: hsl(220 20% 14%);

		width: var(--width);
		color: var(--todos-text);
		background-color: var(--todos-bg);
		border-radius: var(--radius-base);
		border: 1px solid var(--color-gray-90);
		box-shadow: 0 0 4px var(--shadow-1);
	}

	.todo-list {
		list-style: none;
	}

	.actions {
		position: relative;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: var(--spacing-8) var(--spacing-16);
		font-size: 0.9rem;
		border-top: 1px solid var(--color-gray-90);
	}

	.actions:before {
		content: '';
		height: 40px;
		position: absolute;
		right: 0;
		bottom: 0;
		left: 0;
		box-shadow:
			0 1px 1px hsla(0, 0%, 0%, 0.2),
			0 8px 0 -3px hsl(0, 0%, 96%),
			0 9px 1px -3px hsla(0, 0%, 0%, 0.2),
			0 16px 0 -6px hsl(0, 0%, 96%),
			0 17px 2px -6px hsla(0, 0%, 0%, 0.2);
		z-index: -1;
	}
</style>
