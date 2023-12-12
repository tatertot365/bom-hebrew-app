<script>
import bomJSON from '../json/book-of-mormon.json';

/**
 * @type {{ word: any; reverse_translation: any; }[]}
 */
let additionalWords;
/**
 * @type {{ reference: string; text: string; verse: number; }[]}
 */
let bomWords;
/**
 * @type {{ trans: any; dict: any; }}
 */
let parsedData;
/**
 * @type {never[]}
 */
let wordTerms;
/**
 * @type {any[]}
 */
 let accordionStates = [];

let translate_word = '';
let transWord = '';
let partOfSpeech = '';

async function findWordInBOM() {
    bomWords = [];
    bomJSON.books.forEach(book => {
        book.chapters.forEach(chapter => {
            chapter.verses.forEach(verse => {
                if (verse.text.includes(translate_word)) {
                    bomWords.push(verse);
                }
            });
        });
    });
}

/**
 * @param {{ trans: any; dict?: any; }} json
 */
function parseTranslationJSON(json) {
    transWord = json.trans;
    partOfSpeech = json.dict[0].pos;
    wordTerms = json.dict[0].terms;

    additionalWords = [];

    for (let i = 0; i < json.dict[0].entry.length; i++) {
        let newWord = {}
        newWord['word'] = json.dict[0].entry[i].word;
        newWord['reverse_translation'] = json.dict[0].entry[i].reverse_translation;

        additionalWords.push(newWord);
    }

}

async function translate() {
    const url =
        import.meta.env.VITE_TRANSLATE_URL;
    const options = {
        method: 'POST',
        headers: {
            'content-type': 'application/x-www-form-urlencoded',
            'X-RapidAPI-Key': import.meta.env.VITE_RAPID_API_KEY,
            'X-RapidAPI-Host': import.meta.env.VITE_RAPID_API_HOST
        },
        body: new URLSearchParams({
            from: 'auto',
            to: 'he',
            text: translate_word
        })
    };

    try {
        const response = await fetch(url, options);
        const result = await response.text();
        parsedData = JSON.parse(result);
        parseTranslationJSON(parsedData);
        await findWordInBOM();
    } catch (error) {
        console.error(error);
    }
}

/**
 * @param {number} index
 */
function toggleAccordion(index) {
  accordionStates[index] = !accordionStates[index];
}
</script>

<main class="container mt-5">
    <form on:submit|preventDefault={translate} class="mb-4">
        <label for="textInput" class="form-label center-text text-center">
            Enter a word and see its Hebrew translation and where it is found in the Book of Mormon, <br /> verses from the Book of Mormon will only appear if the word is entered in English. <br /> (It is case sensitve)
        </label>
        <input type="text" id="textInput" bind:value={translate_word} class="form-control display-center" style="max-width: 50%;"/>

        <div class="d-flex justify-content-center">
            <button type="submit" class="btn btn-primary mt-2">Submit</button>
        </div>
    </form>
    <div class="row">
        <div class="col-lg-6 d-flex flex-column align-items-center">
            {#if transWord}
            <p class="lead">Translated word: {transWord}</p>
            {/if}
            {#if partOfSpeech}
            <p class="lead">Part of speech: {partOfSpeech}</p>
            {/if}
            {#if wordTerms}
            <p class="lead">Additional translations:</p>
            <ul class="list-unstyled">
                {#each wordTerms as word}
                <li>{word}</li>
                {/each}
            </ul>
            {/if}
            {#if additionalWords}
            <p class="lead">Additional meanings:</p>
            <ul class="list-unstyled">
                {#each additionalWords as word}
                <li>{word.word}</li>
                <ul>
                    {#each word.reverse_translation as translation}
                    <li>{translation}</li>
                    {/each}
                </ul>
                {/each}
            </ul>
            {/if}
        </div>
        <div class="col-lg-6 d-flex flex-column align-items-center">
          {#if bomWords}
            <p class="lead">Found in the Book of Mormon:</p>
            <ul class="list-unstyled">
              {#each bomWords as { reference, text }, index}
                <li>
                  <button on:click={() => toggleAccordion(index)} style="cursor: pointer; margin-top: 10px; margin-bottom: 10px;" class="btn btn-primary">
                    {reference}
                  </button>
                  {#if accordionStates[index]}
                    <div class="accordion-content">
                      {text}
                    </div>
                  {/if}
                </li>
              {/each}
            </ul>
          {/if}
        </div>
    </div>
</main>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">

<style>
.center-text {
    display: flex;
    justify-content: center;
    align-items: center;
}

.accordion-content {
    display: block;
    padding: 10px;
    margin-top: 10px;
    margin-bottom: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
}

.display-center {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
</style>
