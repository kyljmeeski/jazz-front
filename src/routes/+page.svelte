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

		const allowedVowels = ['ө', 'ү', 'а', 'о', 'е', 'и', 'ы', 'э', 'у', 'я', 'ё', 'ю']; // Разрешённые гласные
		const longVowelMapping = {
			'у': 'уу',
			'а': 'аа',
			'э': 'ээ',
			'ө': 'өө',
			'ү': 'үү',
			'о': 'оо'
		};

		let wordsMap = {
			'зат': [
				['адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'адам', 'абад'],
				['кадам', 'кабат']
			],
			'сын': [
				['жаман']
			]
		};

		// Функция для вставки буквы в соответствующее поле и проверки состояния чекбокса
		function insertVowel(index, vowel) {
			vowels[index] = vowel;

			// Проверка возможности удлинения вставленной буквы
			if (longVowelMapping[vowel]) {
				longVowelStates[index] = true;
			} else {
				longVowelStates[index] = false;
			}
		}

		// Ограничение ввода только гласными буквами
		function handleVowelInput(event, index) {
			const value = event.target.value.toLowerCase();
			if (allowedVowels.includes(value)) {
				vowels[index] = value; // Если это гласная буква, сохранить её

				// Активировать чекбокс, если введена гласная, которую можно удлинить
				longVowelStates[index] = !!longVowelMapping[value];
			} else {
				event.target.value = ''; // Удалить все другие символы
			}
		}

		// Функция для замены гласной на долгую
		function toggleLongVowel(event, index) {
			const isChecked = event.target.checked;
			const currentVowel = vowels[index];

			// Если чекбокс активен и текущая гласная допустима для долготы
			if (isChecked && longVowelMapping[currentVowel]) {
				vowels[index] = longVowelMapping[currentVowel];
			} else if (!isChecked && Object.values(longVowelMapping).includes(currentVowel)) {
				// Если чекбокс выключен, возвращаем краткую версию гласной
				vowels[index] = Object.keys(longVowelMapping).find(key => longVowelMapping[key] === currentVowel);
			}
		}

		// Функция для создания объекта
		async function createObject() {
			const headless = !withHead;
			const tailless = !withTail;
			const result = {
				'vowels': vowels.slice(0, vowelsCount), 'consonants': [], headless, tailless, 'first-letter': firstLetter
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
					wordsMap = await response.json();
					console.log(wordsMap);
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
				<button on:click={() => firstLetter='ө'}>ө</button>
				<button on:click={() => firstLetter='ү'}>ү</button>
				<button on:click={() => firstLetter='ң'}>ң</button>
			</div>

			{#each Array(vowelsCount) as _, index}
				<div class="letter">
					<input type="text" maxlength="1" bind:value={vowels[index]} on:input={(e) => handleVowelInput(e, index)} />
					<input type="checkbox" on:change={(e) => toggleLongVowel(e, index)} disabled={!longVowelStates[index]} />
					<button on:click={() => insertVowel(index, 'ө')}>ө</button>
					<button on:click={() => insertVowel(index, 'ү')}>ү</button>
				</div>
			{/each}

			<button on:click={createObject}>Кана?</button>
		</section>
		<section class="response">
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
	</style>
