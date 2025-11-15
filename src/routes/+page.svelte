<script>
	/* function loginWithGoogle() {
		console.log("Logging in with Google...");

	} */


let activeTab = "/";
 function goTo(path) { 
  window.location.href = path;
 }	
 
</script>

<div class="login-container">
	<div class="login-box">
		<div class="logo-wrapper">
			<img src="/logo.png" alt="Logo" class="logo" />
		</div>
		<h2>S.A.T.O ADMIN</h2>
		<p>SIGN-IN</p>

		<button class="google-btn" class:active={activeTab === "/"}	
		on:click={ () => goTo("library")}>
		
		<img src="https://www.svgrepo.com/show/475656/google-color.svg" alt="Google" class="google-icon" />
			<span>Continue using Google</span>
		</button>
	</div>
</div>

<style>
	/* Whole background */
	.login-container {
		background-color: #19333c;
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	/* Center box */
	.login-box {
		background: #122831;
		padding: 3rem 4rem;
		border-radius: 15px;
		box-shadow: 0 0 25px rgba(0, 0, 0, 0.427);
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
		position: relative;
		min-width: 320px;
		border: 1px solid rgb(69, 120, 165);
	}

	/* Logo circle floating */
	.logo-wrapper {
		position: absolute;
		top: -160px;
		border-radius: 50%;
		border: 3px solid #1e4b3a;
		box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
        width: 250px;
		height: 250px;
	}

	.logo {
		width: 250px;
		height: 250px;
		border-radius: 50%;
		object-fit: cover;
	}

	h2 {
		margin-top: 60px; /* space under floating logo */
		color: white;
		font-weight: 700;
	}

	p {
		color: #b9e4d3;
		margin-bottom: 0.5rem;
	}

	.google-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 0.8rem;
		background-color: black;
		color: white;
		border: none;
		padding: 0.8rem 1.5rem;
		border-radius: 10px;
		font-size: 0.95rem;
		cursor: pointer;
		transition: all 0.2s ease;
	}

	.google-btn:hover {
		background-color: #0c0c0c;
		transform: scale(1.03);
	}

	.google-icon {
		width: 22px;
		height: 22px;
	}
</style>
