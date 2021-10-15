<!--Bad code, don't judge pls-->
<script>
	let constants = {
		"king_of_rap":"King of Rap",
		"queen_of_soul":"Queen of Soul",
		"king_of_jazz":"King of Jazz",
		"queen_of_pop":"Queen of Pop",
		"king_of_rnb":"King of RnB",
		"queen_of_christmas":"Queen of Christmas",
		"king_of_antipop":"King of AntiPop",
		"king_of_reggae":"King of Reggae",
		"king_of_folk":"King of Folk",
		"queen_of_hiphop":"Queen of HipHop",
		"queen_of_jazz":"Queen of Jazz",
		"queen_of_disco":"Queen of Disco",
		"king_of_youtube":"King of YouTube",
		"king_of_rocknroll":"King of Rock n' Roll"
	}
	let baseURL = 'http://localhost:8888/'
	//On load
	let questionDataPromise = getQuestion();
	let currentQuestionDiv;
	let answerDataPromise;
	let optionSelected;
	let formErrorMessage = '';
	
	async function submitAnswer(requestData) {
		console.log(JSON.stringify(requestData))
		let response = await fetch(baseURL + 'answer', {
			method: 'POST',
			headers: {'Content-Type': 'text/plain'},
			body: JSON.stringify(requestData)
		});
		let result = await response.json();
		return result;
	}

	async function getQuestion() {
		let response = await fetch(baseURL + 'question');
		if (response.ok) {
			let json = await response.json();
			return json;
		} else {
			throw new Error(response.status);
		}
	}

	function handleSubmit() {
		if (!optionSelected) {
			formErrorMessage = "Select at least one option";
			return;
		}
		let requestData = {
			question: currentQuestionDiv.innerText,
			option: optionSelected,
			feedback: feedback
		}
		answerDataPromise = submitAnswer(requestData);
		optionSelected = null;
		formErrorMessage = '';
		isQuestion = !isQuestion;
	}

	function handleNext() {
		questionDataPromise = getQuestion();
		isQuestion = !isQuestion;
	}

	let isQuestion = true;
	let feedback;
</script>

<body>
	<div id="top-bar">👑The royals of <i>x</i> quiz👑</div>

	<div id="middle-pane">
		{#if isQuestion}
			{#await questionDataPromise}
				<span class="info-text">...loading</span>
			{:then data}
				<div id="question-container">
					<div id="question-id" class="hidden" bind:this={currentQuestionDiv}>{data.question}</div>
					<div id="question">Who is the {constants[data.question]}?</div>
					<div id="options">
						{#each data.options as name}
							<label>
								<input type="radio" bind:group={optionSelected} name="optionSelected" value={name} checked>
								{name}
							</label>
						{/each}
					</div>
				</div>
			{:catch error}
				<span class="info-text">...oops, error occurred. Contact support with logs.
				{error}</span>
			{/await}
			<div id="error-message">{formErrorMessage}</div>
			<div id="feedback">
				Feedback:<br>
				<textarea rows="4" cols="50" bind:value={feedback}></textarea>
			</div>
			<button preventDefault id="submit-btn" on:click={() => {handleSubmit()}}>Submit!</button>
		{:else}
			{#await answerDataPromise}
				<span class="info-text">...loading</span>
			{:then data}
				<div id="king-image__title">Pictured: {constants[data.image]}</div>
				<div id="king-image">
					<img class="king-image__content" src={data.imageData} alt={data.image}/>
				</div>
				<div id="message">{data.message}</div>
			{:catch error}
				<span class="info-text">...oops, error occurred. Contact support with logs.
				{error}</span>
			{/await}
			<button preventDefault id="next-btn" on:click={() => {handleNext()}}>
				Next question
			</button>
		{/if}
	</div>






	<div id="footer">
		Made with ❤️ and 🤦 · <a href="https://gist.github.com/CatalanCabbage/381acb9f6819b9cb62f5a1807beea003#what-is"><i>View on GitHub</i></a>
	</div>
</body>

<style>
	body {
		overflow: hidden;
	}
	#top-bar {
		text-align: center;
		font-family: 'AnotherDayV2';
		color: lightblue;
		background: repeating-radial-gradient(
			circle,
			purple,
			purple 10px,
			#4b026f 10px, 
			#4b026f 20px
		);
		font-size: 3em;
		padding: 0.5em;
	}
	#question-container {
		background-color: rgb(75, 0, 146);
		border-radius: 28px;
		background: #4b0090;
		box-shadow:  27px 27px 51px #3e0076,
					-27px -27px 51px #5900aa;
		padding-bottom: 2em;
		padding: auto 1em;
		border-radius: 5px;
		width: fit-content;
		color: white;
	}
	#question {
		font-size: 2em;
		margin: 1em;
		font-weight: bold;
	}
	#options {
		font-size: 1.5em;
	}
	#middle-pane {
		background-color: rgb(75, 0, 146);
		text-align: center;
		height: 90%;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
	#submit-btn {
		bottom: 2em;
	}
	#error-message {
		margin: 0.5em;
		color: aliceblue;
	}
	.king-image__content {
		width: 25em;
		height: 25em;
		object-fit: cover;
		
	}
	#king-image {
		box-shadow:  27px 27px 51px #3e0076,
					-27px -27px 51px #5900aa;
	}
	#king-image__title {
		margin-top: -2.5em;
		font-size: 1.2em;
		font-style: italic;
		padding: 0.5em;
		color: azure;
		opacity: 0.6;
	}
	#message {
		font-size: 2em;
		font-weight: bold;
		margin: 0.5em;
		color: beige;
	}
	#feedback {
		color: antiquewhite;
	}
	button {
		padding: 1em;
	}



	.hidden {
		opacity: 0.01;
	}
	#footer {
		position: absolute;
		bottom: 0;
		background-color: lightcyan;
		width: 100%;
		text-align: center;
		opacity: 0.8;
		padding: 0.5em;
	}
	.info-text {
		color: white;
	}
</style>