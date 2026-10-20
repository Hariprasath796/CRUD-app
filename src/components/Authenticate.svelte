<script>
	import { authHandlers } from "../store/store";

    let email = $state('');
    let password = $state(''); 
    let confirmPassword = $state('');
    let error = $state(false);
    let register = $state(true);
    let authenticating = $state(false);
    async function handleAuthenticate(){
        if(authenticating)
            {
                return;
            }
                
        if(!email || !password || (register && !confirmPassword)){
            error = true;
            return;
        }
        authenticating = true;
        try {
            if (!register) {

                await authHandlers.login(email, password);

            } else {

                await authHandlers.signup(email, password);

            }

        } catch (err) {

            console.log("There was an auth error", err);

            error = true;

        }finally{
            authenticating = false;
        }
    }

    function handleRegister(){
        register = true;
    }

    function handleLogin(){
        register = false;
    }
</script>

<div class="flex flex-col items-center justify-center flex-1 p-6">
    <form class="flex flex-col gap-5 w-[400px] max-w-full mx-auto">
        
        <h1 class="text-3xl font-bold flex items-center flex-col text-black">
            {register ? "Register" : "Login"}
        </h1>
        {#if error}
        <p class="text-red-500 text-sm text-center ">The information you have entered is not correct.</p>
        {/if}
        
        <label class="border border-gray-600 rounded-xl">
            <input
                class="w-full bg-transparent text-black px-4 py-2 outline-none"
                type="email"
                placeholder="Email"
                bind:value={email}
            />
        </label>

        <label class="border border-gray-600 rounded-xl">
            <input
                class="w-full bg-transparent text-black px-4 py-2 outline-none"
                type="password"
                placeholder="Password must be more than 6 characters"
                bind:value={password} 
            />
        </label>
        {#if register}
        <label class="border border-gray-600 rounded-xl">
            <input
                class="w-full bg-transparent text-black px-4 py-2 outline-none"
                type="password"
                placeholder="Confirm Password must be more than 6 characters"
                bind:value={confirmPassword}
            />
        </label>
        {/if}
        <button
            class="flex items-center justify-center bg-black hover:bg-gray-600  text-white font-semibold py-2 rounded-xl transition hover:cursor-pointer"
            type="submit"
            on:click={handleAuthenticate}>
            {#if authenticating}
            <i class="fa-solid fa-spinner animate-spin"></i>
            {:else}
            Submit
            {/if}
        </button>
    </form>
    <div class=" px-3.5 py-0 overflow-hidden text-center w-[400px] max-w-full my-0 mx-auto ">
        <p class="relative text-black
        before:absolute before:content-['']
        before:top-1/2 before:left-full
        before:-translate-y-1/2
        before:w-screen before:h-[1.5px]
        before:bg-black

        after:absolute after:content-['']
        after:top-1/2 after:right-full
        after:-translate-y-1/2
        after:w-screen after:h-[1.5px]
        after:bg-black">OR</p>
        {#if register}

            <div class="flex items-center gap-2 justify-center">

                <p class="text-black">
                    Already have an account?
                </p>

                <button
                    type="button"
                    class="text-black hover:underline cursor-pointer"
                    on:click={handleLogin}
                >
                    Login
                </button>
            </div>

        {:else}

            <div class="flex items-center gap-2 justify-center">

                <p class="text-black">
                    Don't have an account?
                </p>

                <button
                    type="button"
                    class="text-black hover:underline cursor-pointer"
                    on:click={handleRegister}
                >
                    Register
                </button>
            </div>

        {/if}
    </div>
</div>
