<!-- <GoogleAnalytics ga={ga} trackingId={'UA-156429702-1'}/> -->
<div
  class={`container autocomplete column ${($advancedSearch && !$autocompleteBypass) ? "is-10" : "is-8"}`}
  style="margin-top: -15px;"
>
    <form
       class="columns is-vcentered is-multiline"
      on:submit|preventDefault={handleSubmit}
    >
      {#if !$autocompleteBypass}
        <div class="column is-1 is-hidden-mobile">
          <button
            type="reset"
            class="button is-size-5 is-info"
            on:click|preventDefault={ toggleAdvancedSearch }
            title={ $advancedSearch ? "retour à la recherche simple" : "activer la recherche avancée" }
          >
            <FontAwesomeIcon icon={ $advancedSearch ? faMinus : faPlus }/>
          </button>
        </div>
      {/if}

      <div class={`column ${$autocompleteBypass ? "is-11" : "is-9" }`}>
          <div class="columns is-multiline is-mobile">
            {#each inputsKeys as key}
              {#if isActive[key]}
                {#if $searchInput[key].section}
                  <span class="column is-size-5 is-2 is-mobile-fullwidth is-mobile-centered is-left">{$searchInput[key].section}</span>
                {/if}
                <input
                  autoComplete="off"
                  placeholder={$searchInput[key].placeholder}
                  class={`is-size-5 column is-${$searchInput[key].size} ${isValid(key) ? "" : "is-danger"}`}
                  bind:value={$searchInput[key].value}
                  title={$searchInput[key].title}
                  on:input={() => handleInput(key)}
                  on:blur={focusInput(key,false)}
                  on:focus={focusInput(key,true)}
                  disabled={$searchInput[key].disabled}
                />
              {/if}
            {/each}
          </div>
      </div>
      {#if !$autocompleteBypass}
        <div class="column is-2">
          <div class="columns is-mobile">
            <div class="column is-desktop12 is-mobile-6">
              <button
                type="submit"
                class="button is-size-5 is-fullwidth is-info"
              >
                Recherche
              </button>
            </div>
            <div class="column is-6 is-hidden-desktop">
              <button
                type="reset"
                class="button is-size-5 is-info is-fullwidth"
                on:click|preventDefault={ toggleAdvancedSearch }
                title={ $advancedSearch ? "retour à la recherche simple" : "activer la recherche avancée" }
              >
                <FontAwesomeIcon icon={ $advancedSearch ? faMinus : faPlus }/>
              </button>
            </div>
          </div>
        </div>
        <Autocomplete/>
      {:else}
        <div class="column is-1">
          <button
            type="reset"
            class="button is-fullwidth is-size-5 is-info small-margin-mobile"
            on:click|preventDefault={ toggleAdvancedSearch }
            title={ $advancedSearch ? "retour à la recherche simple" : "activer la recherche avancée" }
          >
            <FontAwesomeIcon icon={ $advancedSearch ? faMinus : faPlus }/>
          </button>
        </div>
      {/if}
      {#if $advancedSearch}
        <div class="column is-12 is-size-5">
          <div
            class="field small-margin-mobile"
            style="margin-bottom:-1rem!important;"
            on:click|preventDefault={ toggleFuzzySearch }
          >
            <label for="switchRoundedInfo">recherche floue &nbsp;</label>
            <input id="switchRoundedInfo" type="checkbox" name="switchRoundedInfo" class="switch is-rounded is-info is-unchecked-danger" bind:checked={$fuzzySearch}>
            <label for="switchRoundedInfo"></label>
          </div>
        </div>
      {/if}
    </form>
</div>
{#if infoDisplay}
  <div
    class="info-footer"
    on:click={infoDisplay=false}
  >
      <p><FontAwesomeIcon icon={faQuestionCircle} class="is-lower"/></p>
      <p>
      {#each inputsKeys as key}
        {#if $searchInputFocus[key] && $searchInputFocus[key].focus}
          {$searchInput[key].title}
        {/if}
      {/each}
      </p>
  </div>
{/if}

<script>
  import FontAwesomeIcon from './FontAwesomeIcon.svelte'

  import { advancedSearch, searchInput, searchCanvas, autocompleteBypass, infoDisplayOption, autocompleteResults, autocompleteDisplay, searchInputFocus, searchTyping, fuzzySearch } from '../tools/stores.js';
  import { search, searchString, searchAutocompleteTrigger, searchSubmit, searchURLUpdate, toggleAdvancedSearch, toggleFuzzySearch } from '../tools/search.js';
  import Autocomplete from './Autocomplete.svelte';

  import {
      faMinus,
      faPlus,
      faQuestionCircle
  } from '@fortawesome/free-solid-svg-icons';

  let gtag;
  const gtagFail = () => {console.log("GA not loaded")}

  $: gtag = window.gtag || gtagFail;

  let lastInput = {}

  let inputsKeys;

  let infoDisplay;

  $: inputsKeys = Object.keys($searchInput)

  $: $autocompleteDisplay=Object.keys($searchInputFocus).some(key => $searchInputFocus[key].focus);
  $: infoDisplay=$infoDisplayOption && Object.keys($searchInputFocus).some(key => $searchInputFocus[key].focus);

  let isActive;

  $: isActive = Object.keys($searchInput).map((key) => {
    let path = $searchInput[key].path ? $searchInput[key].path.replace(/\..*/,"") : undefined
    let subPath = $searchInput[key].path ? $searchInput[key].path.replace(/(^[^\.]*$|.*\.(.*)$)/,"$2") : ""
    subPath = subPath ?
                ( path ?
                  ( ( $searchCanvas[path] && $searchCanvas[path][subPath] ) ?
                      $searchCanvas[path][subPath].active && $searchInput[key].active
                      : false )
                  : false )
              : true
    path = path ? ( $searchCanvas[path] ? $searchCanvas[path].active : false ) : true
    return [ key, path && subPath ]
  }).reduce((a, b) => {
    a = Array.isArray(a) ? { [a[0]]: a[1] } : a;
    a[b[0]]=b[1];
    return a;
  }

  );

  const focusInput = (key, value) => {
    setTimeout(() => {
      searchInputFocus.update(v => {
        v[key]={ focus : value };
        return v
      })
    }, 300);
  }

  const handleSubmit = () => {
    searchSubmit();
    searchURLUpdate();
    // gtag('config', 'UA-156429702-1');
    gtag('event', 'button', {
      event_category: 'recherche',
      event_label: searchString($searchInput)
    });
  }

  const startDate = new Date().getTime();

  const date = () => {
    return Math.round(new Date().getTime() - startDate);
  }

  const isValid = (key) => {
    if ($searchInput[key].mask && $searchInput[key].mask.validation) {
      return $searchInput[key].mask.validation($searchInput[key].value);
    } else {
      return true;
    }
  }

  const handleInput = (key) => {
    if ($searchInput[key].mask && $searchInput[key].mask.typing) {
      if (!$searchInput[key].mask.typing($searchInput[key].value)) {
        $searchInput[key].value = lastInput[key] || '';
      }
      lastInput[key] = $searchInput[key].value
    }
    if (isValid(key)) {
      $searchTyping = date() + 350;
      setTimeout(() => {
        if (date() > $searchTyping) {
          if ($autocompleteBypass) {
            handleSubmit();
          } else {
            autocomplete();
          }
        } else {
          console.log("key input limiter")
        } }, 355)
    }
  }

  const autocomplete = async () => {
    if (searchAutocompleteTrigger($searchInput)) {
      const state = await search($searchInput);
      $autocompleteResults = state.results;
      $autocompleteDisplay = ($autocompleteResults.length > 0)
    } else {
      $autocompleteResults = []
      $autocompleteDisplay = false
    }
  }

</script>

<style>

  input {
    height: 2.35em;
    width: 100%;
  }

  input[type="checkbox"], input[type="radio"] {
      vertical-align: baseline;
  }

  button, input, select, textarea {
      margin: 0;
  }

  .container {
    flex-grow: 1;
    margin: 0 auto;
    position: relative;
    width: auto;
  }

  @media screen and (min-width:1024px) {
    .container {
      max-width: 960px;
    }
  }

  @media screen and (min-width:1216px) {
    .container {
      max-width: 1152px;
    }
  }

  @media screen and (min-width:1408px) {
    .container {
      max-width: 1344px;
    }
  }

  .column {
    display: block;
    flex-basis: 0;
    flex-grow: 1;
    flex-shrink: 1;
    padding: .75rem;
  }

  .button {
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    -moz-appearance: none;
    -webkit-appearance: none;
    align-items: center;
    border: 1px solid transparent;
    border-radius: 4px;
    box-shadow: none;
    display: inline-flex;
    font-size: 1rem;
    height: 2.25em;
    line-height: 1.5;
    position: relative;
    vertical-align: top;
    background-color: #fff;
    border-color: #dbdbdb;
    border-width: 1px;
    color: #363636;
    cursor: pointer;
    justify-content: center;
    padding: calc(.375em - 1px) .75em;
    text-align: center;
    white-space: nowrap;
    box-sizing: border-box;
  }

  .button:active,.button:focus {
    outline: 0;
  }

  .button:hover {
    border-color: #b5b5b5;
    color: #363636;
  }

  .button:focus {
    border-color: #3273dc;
    color: #363636;
  }

  .button:focus:not(:active) {
    box-shadow: 0 0 0 .125em rgba(50,115,220,.25);
  }

  .button:active {
    border-color: #4a4a4a;
    color: #363636;
  }

  .button.is-info,.button.is-info:hover {
    background-color: #209cee;
    border-color: transparent;
    color: #fff;
  }

  .button.is-info:hover {
    background-color: #1496ed;
  }

  .button.is-info:focus {
    border-color: transparent;
    color: #fff;
  }

  .button.is-info:focus:not(:active) {
    box-shadow: 0 0 0 .125em rgba(32,156,238,.25);
  }

  .button.is-info:active {
    background-color: #118fe4;
    border-color: transparent;
    color: #fff;
  }

  .button.is-fullwidth {
    display: flex;
    width: 100%;
  }

  @media print,screen and (min-width:769px) {
    .is-size-5 {
      font-size: 1.25rem!important;
    }

    .column.is-1 {
      flex: none;
      width: 8%;
    }

    .column.is-1-5 {
      flex: none;
      width: 12.5%;
    }

    .column.is-2 {
      flex: none;
      width: 17%;
    }

    .column.is-2-12 {
      flex: none;
      width: 17%;
    }

    .column.is-3 {
      flex: none;
      width: 25%;
    }

    .column.is-3-5 {
      flex: none;
      width: 28.5%;
    }

    .column.is-4 {
      flex: none;
      width: 33%;
    }

    .column.is-5 {
      flex: none;
      width: 42%;
    }

    .column.is-6 {
      flex: none;
      width: 50%;
    }

    .column.is-7 {
      flex: none;
      width: 58%;
    }

    .column.is-8 {
      flex: none;
      width: 67%;
    }

    .column.is-9 {
      flex: none;
      width: 75%;
    }

    .column.is-11 {
      flex: none;
      width: 92%;
    }

    .column.is-12,
    .column.is-desktop-12 {
      flex: none;
      width: 100%;
    }
  }

  .is-left {
    text-align: left!important;
  }

  .columns {
    margin-left: -.75rem;
    margin-right: -.75rem;
    margin-top: -.75rem;
  }

  .columns:last-child {
    margin-bottom: -.75rem;
  }

  .columns:not(:last-child) {
    margin-bottom: .75rem;
  }

  .columns.is-mobile {
    display: flex;
  }

  .columns.is-multiline {
    flex-wrap: wrap;
  }

  .columns.is-vcentered {
    align-items: center;
  }

  @media print,screen and (max-width:768px) {
  .is-size-5 {
    font-size: 1rem!important;
  }

  .info-footer {
    color: #209cee;
    z-index: 40;
    width: 100%;
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    padding: 5px;
    background: rgba(255, 255, 255, 0.8);
    border-top: solid 1px #209cee;
    vertical-align: center;
    justify-content: center;
    display: inline-block;
  }

    .small-margin-mobile {
      margin-top: 0.75rem!important;
      margin-bottom:-1rem!important;
    }

    [placeholder]{
      text-align:center;
    }

    .is-mobile-centered {
      text-align: center!important;
    }

    .is-hidden-mobile {
      display: none;
    }

  .columns.is-mobile>.column.is-1,
  .columns.is-mobile>.column.is-1-5,
  .columns.is-mobile>.column.is-2,
  .columns.is-mobile>.column.is-3,
  .columns.is-mobile>.column.is-3-5,
  .columns.is-mobile>.column.is-4,
  .columns.is-mobile>.column.is-5,
  .columns.is-mobile>.column.is-6,
  .columns.is-mobile>.column.is-mobile-6,
  .columns.is-mobile>.column.is-7
   {
    flex: none;
    width: 50%;
  }

  .columns.is-mobile>.column.is-8,
  .columns.is-mobile>.column.is-9,
  .columns.is-mobile>.column.is-10,
  .columns.is-mobile>.column.is-11,
  .columns.is-mobile>.column.is-2-12,
  .columns.is-mobile>.column.is-12,
  .columns.is-mobile>.column.is-mobile-fullwidth
  {
    flex: none;
    width: 100%;
  }
  }

  @media print,screen and (min-width:769px) {
    .info-footer {
      display: none;
    }

    .is-hidden-desktop {
      display: none;
    }
    .columns:not(.is-desktop) {
      display: flex;
    }
  }

  .is-danger {
    border: 3px solid hsl(348, 100%, 61%)!important;
    border-radius: 4px;
  }

  .field {
    display: flex;
    justify-content: center;
  }

  .switch[type="checkbox"] {
    outline:0;
    user-select:none;
    display:inline-block;
    position:absolute;
    opacity:0;
  }
  .switch[type="checkbox"]:focus+label::before,
  .switch[type="checkbox"]:focus+label:before,
  .switch[type="checkbox"]:focus+label::after,
  .switch[type="checkbox"]:focus+label:after {
  outline:1px dotted #b5b5b5
  }
  .switch[type="checkbox"][disabled] {
  cursor:not-allowed
  }
  .switch[type="checkbox"][disabled]+label {
  opacity:0.5
  }
  .switch[type="checkbox"][disabled]+label::before,
  .switch[type="checkbox"][disabled]+label:before {
  opacity:0.5
  }
  .switch[type="checkbox"][disabled]+label::after,
  .switch[type="checkbox"][disabled]+label:after {
  opacity:0.5
  }
  .switch[type="checkbox"][disabled]+label:hover {
  cursor:not-allowed
  }
  .switch[type="checkbox"]+label {
  position:relative;
  display:initial;
  font-size:1rem;
  line-height:initial;
  padding-left:3.5rem;
  padding-top:.2rem;
  cursor:pointer
  }
  .switch[type="checkbox"]+label::before,
  .switch[type="checkbox"]+label:before {
  position:absolute;
  display:block;
  top:0;
  left:0;
  width:3rem;
  height:1.5rem;
  border:0.1rem solid transparent;
  border-radius:4px;
  background:#b5b5b5;
  content:''
  }
  .switch[type="checkbox"]+label::after,
  .switch[type="checkbox"]+label:after {
  display:block;
  position:absolute;
  top:.25rem;
  left:.25rem;
  width:1rem;
  height:1rem;
  transform:translate3d(0, 0, 0);
  border-radius:4px;
  background:#fff;
  transition:all 0.25s ease-out;
  content:''
  }
  .switch[type="checkbox"]:checked+label::before,
  .switch[type="checkbox"]:checked+label:before {
  background:#00d1b2
  }
  .switch[type="checkbox"]:checked+label::after {
  left:1.625rem
  }
  .switch[type="checkbox"].is-rounded+label::before,
  .switch[type="checkbox"].is-rounded+label:before {
  border-radius:24px
  }
  .switch[type="checkbox"].is-rounded+label::after,
  .switch[type="checkbox"].is-rounded+label:after {
  border-radius:50%
  }

  .switch[type="checkbox"].is-info:checked+label::before,
  .switch[type="checkbox"].is-info:checked+label:before {
  background:#209cee
  }

  .switch[type="checkbox"].is-unchecked-warning+label::before,
  .switch[type="checkbox"].is-unchecked-warning+label:before {
  background:#ffdd57
  }

  .switch[type="checkbox"].is-unchecked-danger+label::before,
  .switch[type="checkbox"].is-unchecked-danger+label:before {
  background:#ff3860
  }
  .switch[type="checkbox"].is-unchecked-danger.is-outlined+label::before,
  .switch[type="checkbox"].is-unchecked-danger.is-outlined+label:before {
  background-color:transparent;
  border-color:#ff3860 !important
  }
  .switch[type="checkbox"].is-unchecked-danger.is-outlined+label::after,
  .switch[type="checkbox"].is-unchecked-danger.is-outlined+label:after {
  background:#ff3860
  }

  [placeholder]{
      text-overflow:ellipsis;
  }

  *, ::after, ::before {
    box-sizing: inherit;
  }

</style>