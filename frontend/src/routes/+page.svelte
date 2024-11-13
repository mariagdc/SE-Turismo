<script>
	import { Carousel, Card, Button, Badge } from 'flowbite-svelte';
	export const images = [
		{
			alt: 'Cosmic timetraveler',
			src: '/img_ushuaia/DSC_2034.JPG',
			title: 'cosmic-timetraveler-pYyOZ8q7AII-unsplash.com'
		},
		{
			alt: 'Cristina Gottardi',
			src: '/img_ushuaia/DSC_2226.JPG',
			title: 'cristina-gottardi-CSpjU6hYo_0-unsplash.com'
		},
		{
			alt: 'Johannes Plenio',
			src: '/img_ushuaia/DSC_5937.jpg',
			title: 'johannes-plenio-RwHv7LgeC7s-unsplash.com'
		},
		{
			alt: 'Jonatan Pie',
			src: '/img_ushuaia/DSC_7947.JPG',
			title: 'jonatan-pie-3l3RwQdHRHg-unsplash.com'
		},
		{
			alt: 'Mark Harpur',
			src: '/img_ushuaia/1.jpg',
			title: 'mark-harpur-K2s_YE031CA-unsplash'
		},
		{
			alt: 'Pietro De Grandi',
			src: '/img_ushuaia/2.JPG',
			title: 'pietro-de-grandi-T7K4aEPoGGk-unsplash'
		},
		{
			alt: 'Sergey Pesterev',
			src: '/img_ushuaia/3.JPG',
			title: 'sergey-pesterev-tMvuB9se2uQ-unsplash'
		},
		{
			alt: 'Solo travel goals',
			src: '/img_ushuaia/4.JPG',
			title: 'solotravelgoals-7kLufxYoqWk-unsplash'
		},
		{
			alt: 'Solo travel goals',
			src: '/img_ushuaia/5.JPG',
			title: 'solotravelgoals-7kLufxYoqWk-unsplash'
		}
	];

	let conversationStarted = false;
	let loading = false;
	let error = false;
	let question = null;
	let resultado = null;
	let userResponse = null;

	const restart = () => {
		conversationStarted = false;
		loading = false;
		error = false;
		question = null;
		resultado = null;
		userResponse = null;
	};

	const startConversation = async () => {
		loading = true;
		error = null;
		conversationStarted = true;
		try {
			const response = await fetch('http://127.0.0.1:8000/comenzar');
			const data = await response.json();
			question = data.pregunta;
		} catch (error) {
			error = 'Error al iniciar la conversación.';
		} finally {
			loading = false;
		}
	};

	const fetchNextQuestion = async () => {
		loading = true;
		error = null;
		try {
			const response = await fetch('http://127.0.0.1:8000/consulta', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({ response: userResponse })
			});
			const data = await response.json();
			if (data.pregunta) {
				question = data.pregunta;
			} else {
				question = null;
				resultado = data;
			}
		} catch (error) {
			error = 'Error al iniciar la conversación.';
		} finally {
			loading = false;
		}
	};

	const handleUserResponse = (response) => {
		userResponse = response;
		fetchNextQuestion();
	};

	const scaleAnimation = (x) => scale(x, { duration: 500, easing: quintOut });
</script>

<div class="carouselContainer">
	<Carousel {images} transition={scaleAnimation} duration="3000"></Carousel>
</div>
<div class="innerContainer">
	<Card class="containerCard" size="md">
		<h5 class="cardTitle mb-2 text-2xl font-bold tracking-tight text-gray-900 dark:text-white">
			Chatbot Sistema Experto-Turismo
		</h5>
		<div class="cardContent">
			{#if !conversationStarted}
				<div>
					<Button on:click={startConversation} class="w-fit">Iniciar Conversación</Button>
				</div>
			{:else if loading}
				<p>Cargando...</p>
			{:else if error}
				<p style={{ color: 'red' }}>{error}</p>
			{:else if question}
				<h4 class="cardText mb-3 font-normal leading-tight text-gray-700 dark:text-gray-400">
					{question}
				</h4>
				<div class="cardButtons">
					<Button on:click={handleUserResponse(true)}>Sí</Button>
					<Button on:click={handleUserResponse(false)}>No</Button>
				</div>
			{:else if resultado}
					<h4 class="subtitle is-4">Conversación finalizada. {resultado.resultado}</h4>
					<p class="mb-3 font-normal leading-tight text-gray-700 dark:text-gray-400">
						{resultado.descripcion}
					</p>
					{#if resultado.propiedades.length}
						<p class="mb-3 font-normal leading-tight text-gray-700 dark:text-gray-400">
							Esto se sugirió porque:
						</p>
					{/if}
					<div class="resultTags">
						{#each resultado.propiedades as prop}
						<Badge>{prop}</Badge>
						<!-- <p class="mb-3 font-normal leading-tight text-gray-700 dark:text-gray-400">{prop}</p> -->
						{/each}
					</div>
					<div class="cardButtons">
					<Button on:click={restart} class="w-fit">Reiniciar</Button>
		</div>
			{/if}
		</div>
	</Card>
</div>

<style>
	/* .container {
		position: fixed;
		height: 100vh;
		width: 100vw;
		margin: 0;
		display: flex;
		flex-grow: 1;
		flex-direction: column;
		align-content: center;
		justify-content: center;
	} */

	.carouselContainer {
		position: fixed;
		height: 100vh;
		width: 100vw;
		display: flex;
		flex-direction: column;
	}

	.cardText {
		align-self: center;
	}

	.cardButtons {
		align-self: center;
	}

	.cardContent {
		display: flex;
		flex-direction: column;
		align-self: center;
		align-content: space-around;
		justify-content: space-around;
	}

	.cardTitle {
		align-self: center;
	}

	.resultTags{
		padding-bottom: 15px;
	}

	.innerContainer {
		position: relative;
		flex-direction: row;
		z-index: 9999;
		align-content: flex-start;
		justify-content: center;
		display: flex;
		flex-grow: 1;
	}
</style>
