<script>
	let vowelsCount = 1;
	const vowelsCountOptions = [
		{ value: 1, label: 'Бир муун' },
		{ value: 2, label: 'Эки муун' },
		{ value: 3, label: 'Үч муун' },
	];

	let withHead = false;
	let withTail = false;
	let firstLetter = '';
	let vowels = Array(3).fill(''); // Массив для гласных
	let longVowelStates = Array(3).fill(false); // Состояние для долгих гласных (чекбоксы)
	let consonants = Array(2).fill(''); // Массив для согласных
	let consonantStates = Array(2).fill(false); // Состояние чекбоксов для согласных

	const allowedVowels = ['ө', 'ү', 'а', 'о', 'е', 'и', 'ы', 'э', 'у', 'я', 'ё', 'ю'];
	const longVowelMapping = {
		'у': 'уу',
		'а': 'аа',
		'э': 'ээ',
		'ө': 'өө',
		'ү': 'үү',
		'о': 'оо'
	};

	let wordsMap = { /*... ваш массив слов ...*/ };
	let idioms = []

	function insertVowel(index, vowel) {
		vowels[index] = vowel;
		if (longVowelMapping[vowel]) {
			longVowelStates[index] = true;
		} else {
			longVowelStates[index] = false;
		}
	}

	function insertConsonant(index, consonant) {
		if (consonantStates[index]) {
			consonants[index] = consonant;
		}
	}

	function handleVowelInput(event, index) {
		const value = event.target.value.toLowerCase();
		if (allowedVowels.includes(value)) {
			vowels[index] = value;
			longVowelStates[index] = !!longVowelMapping[value];
		} else {
			event.target.value = '';
		}
	}

	function toggleLongVowel(event, index) {
		const isChecked = event.target.checked;
		const currentVowel = vowels[index];
		if (isChecked && longVowelMapping[currentVowel]) {
			vowels[index] = longVowelMapping[currentVowel];
		} else if (!isChecked && Object.values(longVowelMapping).includes(currentVowel)) {
			vowels[index] = Object.keys(longVowelMapping).find(key => longVowelMapping[key] === currentVowel);
		}
	}

	function handleConsonantInput(event, index) {
		consonants[index] = event.target.value;
	}

	function toggleConsonantState(event, index) {
		consonantStates[index] = event.target.checked;
		if (!consonantStates[index]) {
			consonants[index] = ''; // Если отключено, очищаем значение
		}
	}

	async function createObject() {
		const headless = !withHead;
		const tailless = !withTail;

		// Регулируем массив согласных в зависимости от активных слогов
		const filteredConsonants = consonants
			.map((consonant, index) => consonantStates[index] ? consonant : '') // Очищаем отключенные согласные
			.filter((_, i) => i < vowelsCount - 1); // Оставляем только нужное количество согласных

		const result = {
			'vowels': vowels.slice(0, vowelsCount),
			'consonants': filteredConsonants,
			headless,
			tailless,
			'first-letter': firstLetter
		};

		try {
			const response = await fetch('http://localhost:5000/matching-words', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify(result)
			});
			if (response.ok) {
				let responseJson = await response.json();
				wordsMap = responseJson['words']
				idioms = responseJson['idioms']
				console.log(responseJson);
			}
		} catch (err) {
			console.error(err);
		}
	}
</script>

<main>
	<section class="request">
		<div class="header">
			<input type="checkbox" bind:checked={withHead} />
			<select bind:value={vowelsCount}>
				{#each vowelsCountOptions as option}
					<option value={option.value}>{option.label}</option>
				{/each}
			</select>
			<input type="checkbox" bind:checked={withTail} />
		</div>

		<div class="letter">
			<span>Башы</span>
			<input type="text" maxlength="1" bind:value={firstLetter} />
			<button tabindex="-1" on:click={() => firstLetter='ө'}>ө</button>
			<button tabindex="-1" on:click={() => firstLetter='ү'}>ү</button>
			<button tabindex="-1" on:click={() => firstLetter='ң'}>ң</button>
		</div>

		{#each Array(vowelsCount) as _, index}
			<div class="letter">
				<input type="text" maxlength="1" bind:value={vowels[index]} on:input={(e) => handleVowelInput(e, index)} />
				<input type="checkbox" on:change={(e) => toggleLongVowel(e, index)} disabled={!longVowelStates[index]} />
				<button tabindex="-1" on:click={() => insertVowel(index, 'ө')}>ө</button>
				<button tabindex="-1" on:click={() => insertVowel(index, 'ү')}>ү</button>
			</div>

			{#if index < vowelsCount - 1}
				<div class="letter">
					<input type="text" bind:value={consonants[index]} on:input={(e) => handleConsonantInput(e, index)} disabled={!consonantStates[index]} />
					<input tabindex="-1" type="checkbox" on:change={(e) => toggleConsonantState(e, index)} />
					<button tabindex="-1" on:click={() => insertConsonant(index, 'ң')}>ң</button>
				</div>
			{/if}
		{/each}

		<button on:click={createObject}>Кана?</button>
	</section>

	<section class="response">
		<div>
			<input type="checkbox" id="words-toggle" class="words-toggle">
			<lable for="words-toggle" class="result-type">Сөздөр</lable>
			<div class="words-outer-container">
				{#each Object.entries(wordsMap) as [type, wordsLists]}
					<div class="words-types">
						<input type="checkbox" id={type} class="toggle" />
						<label for={type} class="type">
							{type}
						</label>

						<div class="words-container">
							{#each wordsLists as wordsList}
								<div class="words">
									{wordsList}
								</div>
							{/each}
						</div>
					</div>
				{/each}
			</div>
		</div>
		<div>
			<input type="checkbox" id="idioms-toggle" class="idioms-toggle">
			<lable for="idioms-toggle" class="result-type">Фразеологизмдер</lable>
			<div class="idioms-outer-container result-type">
				{#each idioms as idiom}
					<div>
						{idiom}
					</div>
				{/each}
			</div>
		</div>
	</section>
</main>



	<style>
			main {
          display: flex;
					flex-direction: column;
					gap: 15px;
      }
      section {
          font-family: monospace;
          font-size: 18px;
      }

      section.request {
          width: 200px;
					display: flex;
					flex-direction: column;
					gap: 10px;
			}

			select, input, button {
					font-size: 18px;
					font-family: monospace;
			}

			.header {
					display: flex;
					align-items: center;
					gap: 8px;
			}

			.header select {
					flex-grow: 1;
			}

			.header input[type='checkbox'] {
          width: 15px;
					height: 15px;
			}

			.letter {
					display: flex;
					align-items: center;
					gap: 8px;
					width: 100%;
			}

			.letter input {
					flex-grow: 1;
					min-width: 0;
			}

			.letter input[type='checkbox'] {
          width: 25px;
          height: 25px;
			}

			.response {
          display: flex;
					flex-direction: column;
					gap: 15px;
			}

			.type {
					color: royalblue;
			}

			.words-container {
          display: flex;
					flex-direction: column;
					gap: 5px;
			}

			.words {
          word-break: break-word;
          display: flex;
			}

      .words-container {
          display: none; /* Скрыть контейнер по умолчанию */
      }

      /* Показывать контейнер, если чекбокс выбран */
      .toggle:checked + .type + .words-container {
          display: block;
      }

      .type {
          cursor: pointer;
          padding: 10px; /* Отступы */
      }

      .words {
          padding: 5px 10px; /* Отступы для слов */
      }

			.words-outer-container {
					display: none;
			}

			.words-toggle:checked +.result-type + .words-outer-container {
					display: block;
			}

      .idioms-outer-container {
          display: none;
      }

      .idioms-toggle:checked +.result-type + .idioms-outer-container {
          display: block;
      }
	</style>
