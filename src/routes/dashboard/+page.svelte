<script>
	// @ts-nocheck
	import { onAuthStateChanged } from "firebase/auth";
	import { auth,db } from '$lib/firebase.js';
	import { authHandlers, authStore } from '../../store/store.js';
	import { doc, setDoc, getDoc } from 'firebase/firestore';
	import { get } from 'svelte/store';

	let todolist = $state([]); 
	let currTodo = $state('');
	let error = $state(false);

	let draggedTask = null;

	authStore.subscribe((curr) => {
		todolist = curr?.data?.todos || [];
	});

	function addTodo() {
		error = false;

		if (!currTodo.trim()) {
			error = true;
			return;
		}

		todolist = [
			...todolist,
			{
				id: crypto.randomUUID(),
				text: currTodo,
				status: 'todo'
			}
		];

		currTodo = '';
	}

	function editTodo(taskId) {
		const task = todolist.find((t) => t.id === taskId);

		if (!task) return;

		currTodo = task.text;

		todolist = todolist.filter((t) => t.id !== taskId);
	}

	function removeTodo(taskId) {
		todolist = todolist.filter((t) => t.id !== taskId);
	}

	function dragStart(task) {
		draggedTask = task;
	}

	function allowDrop(event) {
		event.preventDefault();
	}

	function dropTask(targetStatus) {
		if (!draggedTask) return;

		const validMoves = {
			todo: ['doing'],
			doing: ['done'],
			done: ['doing']
		};

		if (!validMoves[draggedTask.status]?.includes(targetStatus)) {
			alert('Invalid Move!');
			draggedTask = null;
			return;
		}

		todolist = todolist.map((task) =>
			task.id === draggedTask.id
				? { ...task, status: targetStatus }
				: task
		);

		draggedTask = null;
	}

	async function saveTodos() {
		try {
			const store = get(authStore);

			const userRef = doc(db, 'users', store.user.uid);

			await setDoc(
				userRef,
				{
					todos: todolist
				},
				{ merge: true }
			);

			console.log('Todos saved successfully');
		} catch (err) {
			console.log('There is an error in saving your information.');
			console.log(err);
		}
	}

	onAuthStateChanged(auth, async (user) => {
		if (user) {
			const userRef = doc(db, "users", user.uid);
			const userSnap = await getDoc(userRef);

			authStore.set({
				user: user,
				loading: false,
				data: userSnap.exists()
					? userSnap.data()
					: { todos: [] }
			});

		} else {
			authStore.set({
				user: null,
				loading: false,
				data: {
					todos: []
				}
			});
		}
	});
</script>

<div class="flex flex-col min-h-screen gap-6 p-6 w-full max-w-7xl mx-auto">
	<div class="flex items-center justify-between">
		<h1 class="text-2xl font-bold text-black">
			Kanban Todo Board
		</h1>
		<h1 class="text-lg font-bold text-gray-700">
			Welcome, <span class="text-black">{get(authStore).user.email}</span>
		</h1>

		<div class="flex items-center gap-4">
			<button
				onclick={saveTodos}
				class="bg-gray-700 text-white px-3 py-3 rounded-sm font-bold hover:opacity-70 cursor-pointer"
			>
				💾 Save
			</button>

			<button
				onclick={authHandlers.logout}
				class="bg-gray-700 text-white px-3 py-3 rounded-sm font-bold hover:opacity-70 cursor-pointer"
			>
				Logout
			</button>
		</div>
	</div>

	<div
		class={
			"flex items-stretch justify-between border border-cyan-600 rounded-md overflow-hidden " +
			(error ? "!border-red-900" : "")
		}
	>
		<input
			bind:value={currTodo}
			type="text"
			placeholder="Enter todo"
			class="w-full bg-transparent p-3.5 flex-1 text-black focus:outline-none"
		/>

		<button
			class="bg-gray-700 text-white px-4 py-3 font-bold hover:opacity-70 cursor-pointer"
			onclick={addTodo}
		>
			Add
		</button>
	</div>

	<div class="grid grid-cols-1 md:grid-cols-3 gap-6 flex-1">

		<!-- TODO -->
		<div
			class="bg-gray-700 rounded-lg p-4 min-h-[500px]"
			ondragover={allowDrop}
			ondrop={() => dropTask('todo')}
		>
			<h2 class="text-xl font-bold text-white mb-4">
				Todo
			</h2>

			{#each todolist.filter(t => t.status === 'todo') as task}
				<div
					draggable="true"
					ondragstart={() => dragStart(task)}
					class="bg-white p-3 rounded mb-3 flex justify-between items-center cursor-move text-black"
				>
					<p>{task.text}</p>

					<div class="flex gap-3">
						<button
							onclick={() => editTodo(task.id)}
						>
							✏️
						</button>

						<button
							onclick={() => removeTodo(task.id)}
						>
							🗑️
						</button>
					</div>
				</div>
			{/each}
		</div>

		<!-- DOING -->
		<div
			class="bg-gray-700 rounded-lg p-4 min-h-[500px]"
			ondragover={allowDrop}
			ondrop={() => dropTask('doing')}
		>
			<h2 class="text-xl font-bold text-white mb-4">
				Doing
			</h2>

			{#each todolist.filter(t => t.status === 'doing') as task}
				<div
					draggable="true"
					ondragstart={() => dragStart(task)}
					class="bg-orange-400 p-3 rounded mb-3 flex justify-between items-center cursor-move"
				>
					<p>{task.text}</p>

					<div class="flex gap-3">
						<button
							onclick={() => editTodo(task.id)}
						>
							✏️
						</button>

						<button
							onclick={() => removeTodo(task.id)}
						>
							🗑️
						</button>
					</div>
				</div>
			{/each}
		</div>

		<!-- DONE -->
		<div
			class="bg-gray-700 rounded-lg p-4 min-h-[500px]"
			ondragover={allowDrop}
			ondrop={() => dropTask('done')}
		>
			<h2 class="text-xl font-bold text-white mb-4">
				Done
			</h2>

			{#each todolist.filter(t => t.status === 'done') as task}
				<div
					draggable="true"
					ondragstart={() => dragStart(task)}
					class="bg-green-400 p-3 rounded mb-3 flex justify-between items-center cursor-move"
				>
					<p>{task.text}</p>

					<div class="flex gap-3">
						<button
							onclick={() => editTodo(task.id)}
						>
							✏️
						</button>

						<button
							onclick={() => removeTodo(task.id)}
						>
							🗑️
						</button>
					</div>
				</div>
			{/each}
		</div>

	</div>
</div>
