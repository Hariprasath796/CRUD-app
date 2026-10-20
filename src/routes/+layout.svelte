<script>
// @ts-nocheck

  import '../app.css';
  import { onMount } from 'svelte';
  import {auth,db} from '../lib/firebase.js';
  import { doc, getDoc, setDoc } from 'firebase/firestore';
  import { authStore } from '../store/store.js';
	import { goto } from '$app/navigation';
  
  const nonAuthRoutes = ['/','/product']
 /** @type {{email?: string | null, todos: string[]}} */ 
  let dataToSetTostore;

  onMount(() => {
	console.log('Mounting');
	const unsubscribe = auth.onAuthStateChanged(async user=>{
	const currentPath = window.location.pathname;
		if(!user && !nonAuthRoutes.includes(currentPath)){
			goto('/');
			return;
		}
		if(user && currentPath === '/'){
			goto('/dashboard')
			return;
		}
		if(!user){
			return;
		}
		try {

			const docRef = doc(db, 'users', user.uid);
			const docSnap = await getDoc(docRef);

			if (!docSnap.exists()) {

				const userRef = doc(db, 'users', user.uid);
				
				dataToSetTostore = {
					email: user.email,
					todos: []
				};

				await setDoc(userRef, dataToSetTostore, {
					merge: true
				});

			} else {
				const userData = docSnap.data();
				dataToSetTostore = userData;
			}

		} catch (err) {

			console.log(err);

		}

		authStore.update((curr) => {
			return {
				...(curr || {}),
				user,
				data: dataToSetTostore,
				loading: false
			};
		});
	});
	return unsubscribe;
  });
  let { children } = $props();
</script>

<div class="flex min-h-screen bg-white relative flex-col text-white">
  {@render children()}
</div>