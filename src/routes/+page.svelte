<script>
  import pokemon from "../lib/data/pokemon.json";
  import { onMount } from "svelte";

  let pokemonList = pokemon;

  let currentPokemon = {
    pokedexId: 1,
    generation: 1,
    category: "Pok\u00e9mon Graine",
    name: {
      fr: "Bulbizarre",
      en: "Bulbasaur",
      jp: "\u30d5\u30b7\u30ae\u30c0\u30cd",
    },
    sprites: {
      regular:
        "https://raw.githubusercontent.com/Yarkis01/PokeAPI/images/sprites/1/regular.png",
      shiny:
        "https://raw.githubusercontent.com/Yarkis01/PokeAPI/images/sprites/1/shiny.png",
      gmax: null,
    },
    types: [
      {
        name: "Plante",
        image:
          "https://raw.githubusercontent.com/Yarkis01/PokeAPI/images/types/plante.png",
      },
      {
        name: "Poison",
        image:
          "https://raw.githubusercontent.com/Yarkis01/PokeAPI/images/types/poison.png",
      },
    ],
    talents: [
      { name: "Engrais", tc: false },
      { name: "Chlorophylle", tc: true },
    ],
    stats: {
      hp: 45,
      atk: 49,
      def: 49,
      spe_atk: 65,
      spe_def: 65,
      vit: 45,
    },
    resistances: [
      { name: "Normal", multiplier: 1 },
      { name: "Plante", multiplier: 0.25 },
      { name: "Feu", multiplier: 2 },
      { name: "Eau", multiplier: 0.5 },
      { name: "\u00c9lectrik", multiplier: 0.5 },
      { name: "Glace", multiplier: 2 },
      { name: "Combat", multiplier: 0.5 },
      { name: "Poison", multiplier: 1 },
      { name: "Sol", multiplier: 1 },
      { name: "Vol", multiplier: 2 },
      { name: "Psy", multiplier: 2 },
      { name: "Insecte", multiplier: 1 },
      { name: "Roche", multiplier: 1 },
      { name: "Spectre", multiplier: 1 },
      { name: "Dragon", multiplier: 1 },
      { name: "T\u00e9n\u00e8bres", multiplier: 1 },
      { name: "Acier", multiplier: 1 },
      { name: "F\u00e9e", multiplier: 0.5 },
    ],
    evolution: {
      pre: null,
      next: [
        { pokedexId: 2, name: "Herbizarre", condition: "Niveau 16" },
        { pokedexId: 3, name: "Florizarre", condition: "Niveau 32" },
      ],
      mega: null,
    },
    height: "0,7 m",
    weight: "6,9 kg",
    egg_groups: ["Monstrueux", "V\u00e9g\u00e9tal"],
    sexe: { male: 87.5, female: 12.5 },
    catch_rate: 45,
    level_100: 1059862,
    forme: null,
  };

  let draggingPokemon = null;
  let lastPokemon = null;
  let mobileLastPokemon = null;
  let mouseX = 0;
  let mouseY = 0;

  let caughtPokemon = Array(100).fill(-1);
  let modifiedPokemon = [];

  let isDraggingMouse = false;
  let isDraggingTouch = false;

  let isHelpCenterOpen = false;
  let creditsOpen = false;

  let currentCase = null;

  let mousedownTime;
  let contextMenuVisible = false;
  let resumeTab = false;

  let typeDropdown = false;

  let typeFilter = {
    plante: {
      name: "Plante",
      etat: false,
    },
    feu: {
      name: "Feu",
      etat: false,
    },
    eau: {
      name: "Eau",
      etat: false,
    },
    electrik: {
      name: "Électrik",
      etat: false,
    },
    glace: {
      name: "Glace",
      etat: false,
    },
    combat: {
      name: "Combat",
      etat: false,
    },
    poison: {
      name: "Poison",
      etat: false,
    },
    sol: {
      name: "Sol",
      etat: false,
    },
    vol: {
      name: "Vol",
      etat: false,
    },
    psy: {
      name: "Psy",
      etat: false,
    },
    insecte: {
      name: "Insecte",
      etat: false,
    },
    roche: {
      name: "Roche",
      etat: false,
    },
    spectre: {
      name: "Spectre",
      etat: false,
    },
    dragon: {
      name: "Dragon",
      etat: false,
    },
    tenebres: {
      name: "Ténèbres",
      etat: false,
    },
    acier: {
      name: "Acier",
      etat: false,
    },
    fee: {
      name: "Fée",
      etat: false,
    },
  };
  let typeFilterOn = [];

  onMount(() => {
    const caughtPokemonData = localStorage.getItem("caughtPokemon");
    const modifiedPokemonData = localStorage.getItem("modifiedPokemon");
    
    const onMouseMove = (event) => {
      if (draggingPokemon) {
        mouseX = event.clientX;
        mouseY = event.clientY;
      }
    };

    if (caughtPokemonData) {
      caughtPokemon = JSON.parse(caughtPokemonData);
    }

    if (modifiedPokemonData) {
      modifiedPokemon = JSON.parse(modifiedPokemonData);
    }

    

    const onMouseUp = () => {
      if (draggingPokemon) {
        const mouseupTime = new Date();
        if (
          mouseupTime - mousedownTime < 400 &&
          caughtPokemon[currentCase] == draggingPokemon
        ) {
          contextMenuVisible = true;
        }
        if (draggingPokemon) {
          if (caughtPokemon[currentCase] !== -1) {
            caughtPokemon = caughtPokemon.map((id) =>
              id === draggingPokemon ? caughtPokemon[currentCase] : id
            );
          } else {
            caughtPokemon = caughtPokemon.map((id) =>
              id === draggingPokemon ? -1 : id
            );
          }
          caughtPokemon[currentCase] = draggingPokemon;
          draggingPokemon = null;
          localStorage.setItem("caughtPokemon", JSON.stringify(caughtPokemon));
        }
      }
      isDraggingMouse = false;
      isDraggingTouch = false;
    };

    window.addEventListener("mousemove", onMouseMove);
    window.addEventListener("mouseup", onMouseUp);
    document.addEventListener("mousedown", function (event) {
      if (
        contextMenuVisible == true &&
        !event.target.closest(".context-menu")
      ) {
        contextMenuVisible = false;
      }
    });

    return () => {
      window.removeEventListener("mousemove", onMouseMove);
      window.removeEventListener("mouseup", onMouseUp);
    };
  });



  
  function getRandomNumber(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }

  function createTypeFilter() {
    typeFilterOn = [];
    if (Object.values(typeFilter).every((value) => value.etat === false)) {
      for (let type in typeFilter) {
        typeFilterOn = [...typeFilterOn, typeFilter[type].name];
      }
    } else {
      for (let type in typeFilter) {
        if (typeFilter[type].etat == true) {
          typeFilterOn = [...typeFilterOn, typeFilter[type].name];
        }
      }
    }
  }

  function pokemonInfoTab(action) {
    if (action == "close") {
      document.querySelector("#pokemon-info").classList.add("hide");
      document.querySelector("main").classList.remove("crop-right");
    } else {
      document.querySelector("#pokemon-info").classList.remove("hide");
      document.querySelector("main").classList.add("crop-right");
    }
  }

  function boitePc(action) {
    if (action == "open") {
      document.querySelector("#boite-pc").style.display = "flex";
      document.querySelector("body").style.overflow = "hidden";
    } else {
      document.querySelector("#boite-pc").style.display = "none";
      document.querySelector("body").style.overflow = "auto";
    }
  }

  function helpCenter(action) {
    if (action == "open") {
      document.querySelector("#help-center").style.display = "flex";
      document.querySelector(".credits-button").style.visibility = "hidden";
      document.querySelector("body").style.overflow = "hidden";
      isHelpCenterOpen = true;
    } else {
      document.querySelector("#help-center").style.display = "none";
      document.querySelector(".credits-button").style.visibility = "visible";
      document.querySelector("body").style.overflow = "auto";
      isHelpCenterOpen = false;
    }
  }

  function credits(action) {
    if (action == "open") {
      document.querySelector("#credits").style.display = "flex";
      document.querySelector(".help-button").style.visibility = "hidden";
      document.querySelector("body").style.overflow = "hidden";
      creditsOpen = true;
    } else {
      document.querySelector("#credits").style.display = "none";
      document.querySelector(".help-button").style.visibility = "visible";
      document.querySelector("body").style.overflow = "auto";
      creditsOpen = false;
    }
  }

  function handleKeydown(event) {
    if (event.key === "Enter") {
      event.target.blur();
    }
  }

  createTypeFilter();

</script>

<header>
  <button
    on:click={() => {
      boitePc("open");
    }}
    class="openPC">Ouvrir la boite PC</button
  >
  <div id="filter" class:dropdown-open={typeDropdown == true}>
    <h3 on:click={() => {
      typeDropdown == true ? typeDropdown = false : typeDropdown = true;;
    }}>Type
      <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="24" height="24" viewBox="0 0 24 24">
        <path d="M 2.65625 6.25 L 1.34375 7.75 L 11.34375 16.75 L 12 17.34375 L 12.65625 16.75 L 22.65625 7.75 L 21.34375 6.25 L 12 14.65625 Z"></path>
        </svg>
    </h3>
    <div class="container list">
      {#each Object.keys(typeFilter) as type}
        <div>
          <input
            type="checkbox"
            id={typeFilter[type].name}
            on:click={() => {
              typeFilter[type].etat = !typeFilter[type].etat;
              createTypeFilter();
            }}
          />

          <label for={typeFilter[type].name}>{typeFilter[type].name}</label>
        </div>
      {/each}
    </div>
  </div>
  <div id="pokemon-info" class="hide">
    <button
      id="close-info"
      class="close-button"
      on:click={() => {
        pokemonInfoTab("close");
      }}
      ><svg
        xmlns="http://www.w3.org/2000/svg"
        x="0px"
        y="0px"
        width="20"
        height="20"
        viewBox="0 0 50 50"
      >
        <path
          d="M 40.783203 7.2714844 A 2.0002 2.0002 0 0 0 39.386719 7.8867188 L 25.050781 22.222656 L 10.714844 7.8867188 A 2.0002 2.0002 0 0 0 9.2792969 7.2792969 A 2.0002 2.0002 0 0 0 7.8867188 10.714844 L 22.222656 25.050781 L 7.8867188 39.386719 A 2.0002 2.0002 0 1 0 10.714844 42.214844 L 25.050781 27.878906 L 39.386719 42.214844 A 2.0002 2.0002 0 1 0 42.214844 39.386719 L 27.878906 25.050781 L 42.214844 10.714844 A 2.0002 2.0002 0 0 0 40.783203 7.2714844 z"
        ></path>
      </svg></button
    >
    <div class="info-tab">
      <img
        src={currentPokemon.sprites.regular}
        alt={`image de ${currentPokemon.name.fr}`}
        class="sprite"
        loading="lazy"
      />
      <div>
        <p class="pokemonId">N°{currentPokemon.pokedexId}</p>
        <h2>{currentPokemon.name.fr}</h2>
        <h3>{currentPokemon.category}</h3>
      </div>
      {#if currentPokemon.pokedexId != 0}
        <div class="container">
          {#each currentPokemon.types as type}
            <div>
              <img src={type.image} alt={type.name} loading="lazy" />
              <h4>{type.name.toUpperCase()}</h4>
            </div>
          {/each}
        </div>
        <div class="container talent-cont">
          <h3>Talents</h3>
          <div class="container">
            {#each currentPokemon.talents as talent}
              {#if talent.tc}
                <div class="talent talent-cache">
                  <p>{talent.name}</p>
                  <img
                    width="20"
                    height="20"
                    src="https://img.icons8.com/material-outlined/48/hide.png"
                    alt="hide"
                  />
                </div>
              {:else}
                <div class="talent">
                  <p>{talent.name}</p>
                </div>
              {/if}
            {/each}
          </div>
        </div>
        <div class="container">
          <div class="single-info">
            <h3>Hauteur</h3>
            <p>{currentPokemon.height}</p>
          </div>
          <div class="single-info">
            <h3>Poids</h3>
            <p>{currentPokemon.weight}</p>
          </div>
        </div>
        <div class="container str-weak">
          <div class="single-info">
            <h3>Faiblesses</h3>
            <div class="container type-list">
              {#each currentPokemon.resistances as weakness}
                {#if weakness.multiplier > 1}
                  <img
                    src={`https://raw.githubusercontent.com/Yarkis01/PokeAPI/images/types/${weakness.name
                      .toLowerCase()
                      .normalize("NFD")
                      .replace(/[\u0300-\u036f]/g, "")}.png`}
                    alt=""
                  />
                {/if}
              {/each}
            </div>
          </div>
          <div class="single-info">
            <h3>Résistances</h3>
            <div class="container type-list">
              {#each currentPokemon.resistances as resistance}
                {#if resistance.multiplier < 1}
                  <img
                    src={`https://raw.githubusercontent.com/Yarkis01/PokeAPI/images/types/${resistance.name
                      .toLowerCase()
                      .normalize("NFD")
                      .replace(/[\u0300-\u036f]/g, "")}.png`}
                    alt=""
                  />
                {/if}
              {/each}
            </div>
          </div>
        </div>
        {#if currentPokemon.evolution != null}
          <div>
            <h3>Évolution</h3>
            <div class="container evolution">
              {#if currentPokemon.evolution.pre}
                {#each currentPokemon.evolution.pre as evolution}
                  <div>
                    <div class="image-cont">
                      <img
                        src={`https://projectpokemon.org/images/sprites-models/bw-animated/${evolution.pokedexId
                          .toString()
                          .padStart(3, "0")}.gif`}
                        alt={evolution.name}
                        loading="lazy"
                      />
                    </div>
                    <p>{evolution.condition}</p>
                  </div>
                {/each}
              {/if}
              <div class="image-cont">
                <img
                  src={`https://projectpokemon.org/images/sprites-models/bw-animated/${currentPokemon.pokedexId
                    .toString()
                    .padStart(3, "0")}.gif`}
                  alt={currentPokemon.name.fr}
                  loading="lazy"
                />
              </div>

              {#if currentPokemon.evolution.next}
                {#if currentPokemon.evolution.pre == null || currentPokemon.evolution.pre.length == 1}
                  {#each currentPokemon.evolution.next as evolution}
                    <div>
                      <p>{evolution.condition}</p>
                      <div class="image-cont">
                        <img
                          src={`https://projectpokemon.org/images/sprites-models/bw-animated/${evolution.pokedexId
                            .toString()
                            .padStart(3, "0")}.gif`}
                          alt={evolution.name}
                          loading="lazy"
                        />
                      </div>
                    </div>
                  {/each}
                {:else}
                  {#each currentPokemon.evolution.next as evolution}
                    <div>
                      <div class="image-cont">
                        <img
                          src={`https://projectpokemon.org/images/sprites-models/bw-animated/${evolution.pokedexId
                            .toString()
                            .padStart(3, "0")}.gif`}
                          alt={evolution.name}
                          loading="lazy"
                        />
                      </div>
                      <p>{evolution.condition}</p>
                    </div>
                  {/each}
                {/if}
              {/if}

              <div class="mega-form">
                {#if currentPokemon.evolution.mega}
                  {#each currentPokemon.evolution.mega as evolution}
                    <div>
                      <p>{evolution.orbe}</p>
                      <div class="image-cont">
                        <img
                          src={evolution.sprites.regular}
                          alt={evolution.name}
                          loading="lazy"
                        />
                      </div>
                    </div>
                  {/each}
                {/if}
              </div>
            </div>
          </div>
        {/if}
        <div>
          <h3>Statistiques</h3>
          <div class="container stats">
            {#each Object.keys(currentPokemon.stats) as stat}
              <div class="single-info">
                <h4>{stat.toUpperCase()}</h4>
                <p>{currentPokemon.stats[stat]}</p>
              </div>
            {/each}
          </div>
        </div>
      {/if}
    </div>
  </div>
  <div class="container popup-button-cont">
    {#if isHelpCenterOpen == false}
      <button
        on:click={() => {
          helpCenter("open");
        }}
        class:active-popup-button={isHelpCenterOpen}
        class="popup-button help-button">?</button
      >
    {:else}
      <button
        on:click={() => {
          helpCenter("close");
        }}
        class:active-popup-button={isHelpCenterOpen}
        class="popup-button help-button">X</button
      >
    {/if}
    {#if creditsOpen == false}
      <button
        on:click={() => {
          credits("open");
        }}
        class:active-popup-button={creditsOpen}
        class="popup-button credits-button">Voir les crédits</button
      >
    {:else}
      <button
        on:click={() => {
          credits("close");
        }}
        class:active-popup-button={creditsOpen}
        class="popup-button credits-button">Fermer</button
      >
    {/if}
  </div>
</header>
<main>
  <div
    on:click={() => {
      credits("close");
    }}
    id="credits"
    class="popup"
  >
    <div id="credits-content" on:click|stopPropagation>
      <h2>Crédits</h2>
      <div>
        <h3>Inspiration</h3>
        <p>Projet Behance de AC1 design : Concept de design pour un site web Pokédex</p>
        <a href="https://www.behance.net/gallery/113562309/Pokemon-Pokedex-Website-Redesign-Concept">Pokemon - Pokedex Website Redesign Concept</a>
      </div>
      <div>
        <h3>Assets</h3>
        <p>Pokéball : <a href="https://codepen.io/raubaca/pen/obaZmG">CSS Animated Poké Ball - CodePen</a></p>
        <p>Icône : <a target="_blank" href="https://icons8.com/icon/85035/hide">Hide</a> icon by <a target="_blank" href="https://icons8.com">Icons8</a></p>
      </div>
      <div>
        <h3>API</h3>
        <p>Les information et les images des Pokémons proviennent de :</p>
        <ul>
          <li>Données Pokémons : Yarkis01 / TyraDex 
            <ul>
              <li><a href="https://github.com/Yarkis01/TyraDex">GitHub - Yarkis01/TyraDex: Une API Pokémon en français. (WIP)</a></li>
              <li><a href="https://tyradex.tech/">Site de TyraDex</a></li>
            </ul>
          </li>
          <li>Sprites style 5G et 3D : <a href="https://projectpokemon.org/">Portal - Project Pokemon Forums</a></li>
          <li>Sprites Pokémon Shuffle : <a href="https://www.pokencyclopedia.info/fr/index.php?id=sprites/spin-off/ico_shuffle">Pokencyclopedia.info, Home page</a></li>
        </ul>
      </div>
      <div>
      <h3>Artists</h3>
      <p>Merci à ces petits artistes d'avoir pris le temps de dessiner et animer les Pokémons de la 6G dans le style de la 5G et de contribuer à la communaité incroyable qu'est celle de Pokémon :</p>
      <ul>
        <li>spyroflame0487</li>
        <li>br0de0</li>
        <li>SkidMarc25</li>
        <li>Creobnil</li>
        <li>N-Kin</li>
        <li>Diegotoon20</li>
        <li>PomPomKing</li>
        <li>OmegalingYT</li>
        <li>MrMudkipss</li>
        <li>Kirbyfan42</li>
        <li>HyperactiveFlummi</li>
        <li>kirbio</li>
        <li>Kattling</li>
        <li>hellfire0raptor</li>
        <li>Snivy101</li>
        <li>ekurepu</li>
        <li>MCH4R1Z4RD</li>
      </ul>
      <p>Source : <a href="https://www.deviantart.com/">DeviantArt</a></p>
    </div>
    </div>
  </div>
  <div
    on:click={() => {
      helpCenter("close");
    }}
    id="help-center"
    class="popup"
  >
    <div id="help-content" on:click|stopPropagation>
      <h2>Comment utiliser le Pokedex ?</h2>
      <p>
        Pour utiliser le Pokedex, il vous suffit de cliquer sur un Pokémon pour
        obtenir des informations sur celui-ci. 
      </p>
      <p>Vous pouvez également filtrer les
        Pokémon par type en cliquant sur les cases à cocher dans la section
        "Type".</p>
      <p>Pour capturer un Pokemon, cliquez sur la pokeball du Pokemon correspondant et 
        il sera transféré automatiquement à la boite PC.</p>
        <div>
          <div>
            <p>Pas capturé</p>
            <div class="pokeball notCaught"></div>
          </div>
          <div>
            <p>Capturé</p>
            <div class="pokeball"></div>
          </div>
        </div>
        <h2>Comment utiliser la boîte PC ?</h2>
        <p>Tous vos Pokémons attrapés se retrouvent dans la boite PC. Pour ouvrir la liste des actions, 
          appuyez brièvement sur un de vos Pokemons. Vous pouvez alors voir son résumé, 
          le relâcher ou bien fermer le menu.</p>

          <p>Le résumé permet de voir votre Pokémon, qui a une chance d'être chromatique (shiny) 
            lors de la capture. Vous pouvez également renommer vos Pokémons</p>

          <p>Pour déplacer un Pokémon, maintenez appuyé un des clics de la souris sur ce dernier
            et relâcher à la position voulu. Sur mobile, restez appuyé sur le Pokémon jusqu'à l'animation et 
            appuyez sur la nouvelle position souhaité.</p>
      </div>
    </div>
  <div
    id="boite-pc"
    on:click={() => {
      boitePc("close");
      resumeTab = false;
    }}
  >
    <div>
      <div class="container">
        <h2>Boite de stockage</h2>
        <button
          class="close-button"
          on:click={() => {
            boitePc("close");
            resumeTab = false;
          }}
          ><svg
            xmlns="http://www.w3.org/2000/svg"
            x="0px"
            y="0px"
            width="20"
            height="20"
            viewBox="0 0 50 50"
          >
            <path
              d="M 40.783203 7.2714844 A 2.0002 2.0002 0 0 0 39.386719 7.8867188 L 25.050781 22.222656 L 10.714844 7.8867188 A 2.0002 2.0002 0 0 0 9.2792969 7.2792969 A 2.0002 2.0002 0 0 0 7.8867188 10.714844 L 22.222656 25.050781 L 7.8867188 39.386719 A 2.0002 2.0002 0 1 0 10.714844 42.214844 L 25.050781 27.878906 L 39.386719 42.214844 A 2.0002 2.0002 0 1 0 42.214844 39.386719 L 27.878906 25.050781 L 42.214844 10.714844 A 2.0002 2.0002 0 0 0 40.783203 7.2714844 z"
            ></path>
          </svg></button
        >
      </div>
      <div id="storage" on:click|stopPropagation on:contextmenu|preventDefault>
        {#each caughtPokemon as _, i}
          <div
            class="storage-case"
            on:mouseenter={() => {
              currentCase = i;
              if (
                isDraggingTouch == true &&
                isDraggingMouse == true &&
                caughtPokemon[currentCase] != -1
              ) {
                caughtPokemon = caughtPokemon.map((id) =>
                  id === mobileLastPokemon ? caughtPokemon[currentCase] : id
                );
                caughtPokemon[currentCase] = mobileLastPokemon;
                draggingPokemon = null;
                localStorage.setItem(
                  "caughtPokemon",
                  JSON.stringify(caughtPokemon)
                );
                isDraggingTouch = false;
              }
            }}
          >
            {#if caughtPokemon[i] != -1}
              {#if contextMenuVisible && lastPokemon == caughtPokemon[i] && draggingPokemon == null}
                <div
                  class="context-menu"
                  style={caughtPokemon.indexOf(lastPokemon) < 10
                    ? "transform: translateY(calc(min(90vh, 90vw) / 10));"
                    : ""}
                >
                  <button
                    style="color: mediumseagreen;"
                    on:click={() => {
                      resumeTab = true;
                      contextMenuVisible = false;
                    }}>Résumé</button
                  >
                  <button
                    on:click={() => {
                      caughtPokemon = caughtPokemon.map((id) =>
                        id === caughtPokemon[i] ? -1 : id
                      );
                      localStorage.setItem(
                        "caughtPokemon",
                        JSON.stringify(caughtPokemon)
                      );
                      modifiedPokemon = modifiedPokemon.filter(
                        (pokemon) => pokemon.id !== lastPokemon
                      );
                      localStorage.setItem(
                        "modifiedPokemon",
                        JSON.stringify(modifiedPokemon)
                      );
                      contextMenuVisible = false;
                      resumeTab = false;
                    }}
                    style="color: firebrick;">Relâcher</button
                  >
                  <button
                    on:click={() => {
                      contextMenuVisible = false;
                    }}>Annuler</button
                  >
                </div>
              {/if}
              <img
                on:contextmenu|preventDefault
                src={`https://www.pokencyclopedia.info/sprites/spin-off/ico_shuffle/ico_shuffle_${pokemonList[
                  caughtPokemon[i]
                ].pokedexId
                  .toString()
                  .padStart(3, "0")}.png`}
                alt={pokemonList[caughtPokemon[i]].name.fr}
                loading="lazy"
                class="sprite"
                class:isDragging={draggingPokemon ==
                  pokemonList[caughtPokemon[i]].pokedexId && isDraggingMouse}
                class:active={pokemonList[caughtPokemon[i]].pokedexId == lastPokemon && contextMenuVisible}
                draggable="false"
                on:mousedown|stopPropagation={() => {
                  draggingPokemon = pokemonList[caughtPokemon[i]].pokedexId;
                  isDraggingMouse = true;
                  lastPokemon = pokemonList[caughtPokemon[i]].pokedexId;
                  mouseX = event.clientX;
                  mouseY = event.clientY;
                  mousedownTime = new Date();
                }}
                on:touchstart={() => {
                  mobileLastPokemon = caughtPokemon[currentCase];
                  draggingPokemon = pokemonList[caughtPokemon[i]].pokedexId;
                  isDraggingTouch = true;
                  setTimeout(() => {
                    if (
                      isDraggingTouch == true &&
                      caughtPokemon[currentCase] == draggingPokemon
                    ) {
                      isDraggingMouse = true;
                    }
                  }, 680);
                  lastPokemon = pokemonList[caughtPokemon[i]].pokedexId;
                  mouseX = event.currentTarget.getBoundingClientRect().left;
                  mouseY = event.currentTarget.getBoundingClientRect().top;
                  mousedownTime = new Date();
                }}
                style={`${
                  draggingPokemon == pokemonList[caughtPokemon[i]].pokedexId
                    ? `left: ${mouseX}px; top: ${mouseY}px;`
                    : ""
                }`}
              />
            {/if}
          </div>
        {/each}
      </div>
    </div>
    {#if resumeTab}
      <div
        id="pokemon-resume"
        on:click|stopPropagation
        on:contextmenu|preventDefault
      >
        <div class="container">
          <h2>Résumé</h2>
          <button
            class="close-button"
            on:mousedown={() => {
              resumeTab = false;
            }}
            ><svg
              xmlns="http://www.w3.org/2000/svg"
              x="0px"
              y="0px"
              width="20"
              height="20"
              viewBox="0 0 50 50"
            >
              <path
                d="M 40.783203 7.2714844 A 2.0002 2.0002 0 0 0 39.386719 7.8867188 L 25.050781 22.222656 L 10.714844 7.8867188 A 2.0002 2.0002 0 0 0 9.2792969 7.2792969 A 2.0002 2.0002 0 0 0 7.8867188 10.714844 L 22.222656 25.050781 L 7.8867188 39.386719 A 2.0002 2.0002 0 1 0 10.714844 42.214844 L 25.050781 27.878906 L 39.386719 42.214844 A 2.0002 2.0002 0 1 0 42.214844 39.386719 L 27.878906 25.050781 L 42.214844 10.714844 A 2.0002 2.0002 0 0 0 40.783203 7.2714844 z"
              ></path>
            </svg></button
          >
        </div>
        <div class="resume-body">
          <h3>
            <input
              type="text"
              maxlength="12"
              value={modifiedPokemon.find(
                (pokemon) => pokemon.id === lastPokemon
              ).name}
              on:blur={() => {
                if (event.target.value.replace(/\s/g, "") == "") {
                  event.target.value = pokemonList[lastPokemon].name.fr;
                }

                modifiedPokemon.find(
                  (pokemon) => pokemon.id === lastPokemon
                ).name = event.target.value;
                localStorage.setItem(
                  "modifiedPokemon",
                  JSON.stringify(modifiedPokemon)
                );
              }}
              on:keydown={handleKeydown}
            />
            <svg
              xmlns="http://www.w3.org/2000/svg"
              x="0px"
              y="0px"
              width="30"
              height="30"
              viewBox="0 0 30 30"
            >
              <path
                d="M24,11l2.414-2.414c0.781-0.781,0.781-2.047,0-2.828l-2.172-2.172c-0.781-0.781-2.047-0.781-2.828,0L19,6L24,11z M17,8	L5.26,19.74c0,0,0.918-0.082,1.26,0.26c0.342,0.342,0.06,2.58,0.48,3s2.644,0.124,2.963,0.443c0.319,0.319,0.297,1.297,0.297,1.297	L22,13L17,8z M4.328,26.944l-0.015-0.007C4.213,26.97,4.111,27,4,27c-0.552,0-1-0.448-1-1c0-0.111,0.03-0.213,0.063-0.313	l-0.007-0.015L4,23l1.5,1.5L7,26L4.328,26.944z"
              ></path>
            </svg>
          </h3>
          <h4>{pokemonList[lastPokemon].name.fr}</h4>
          <div>
            {#if pokemonList[lastPokemon].sprites.shiny != null && modifiedPokemon.find((pokemon) => pokemon.id === lastPokemon).shiny == true}
              <img
                src={pokemonList[lastPokemon].sprites.shiny}
                alt={`image de ${pokemonList[lastPokemon].name.fr}`}
                class="sprite"
                loading="lazy"
              />
            {:else}
              <img
                src={pokemonList[lastPokemon].sprites.regular}
                alt={`image de ${pokemonList[lastPokemon].name.fr}`}
                class="sprite"
                loading="lazy"
              />
            {/if}
            <div class="container">
              {#if pokemonList[lastPokemon].types}
                {#each pokemonList[lastPokemon].types as type}
                  <div>
                    <img
                      class="type"
                      src={type.image}
                      alt={type.name}
                      loading="lazy"
                    />
                    <h4>{type.name.toUpperCase()}</h4>
                  </div>
                {/each}
              {/if}
            </div>
          </div>
        </div>
      </div>
    {/if}
  </div>
  {#each pokemonList as pokemon}
    {#if pokemon.pokedexId != 0}
      {#if pokemon.types[1] != undefined}
        {#if typeFilterOn.indexOf(pokemon.types[0].name) != -1 || typeFilterOn.indexOf(pokemon.types[1].name) != -1}
          <button
            class="pokemon-card"
            on:click={() => {
              currentPokemon = pokemon;
              pokemonInfoTab("show");
            }}
          >
            {#if pokemon.sprites.animated == undefined && pokemon.pokedexId > 650}
              <img
                src={`https://projectpokemon.org/images/normal-sprite/${pokemon.name.en.toLowerCase()}.gif`}
                alt={pokemon.name.fr}
                loading="lazy"
                class="sprite"
              />
            {:else if pokemon.pokedexId > 649 || pokemon.pokedexId == 0}
              <img
                src={pokemon.sprites.animated}
                alt={pokemon.name.fr}
                loading="lazy"
                class="sprite"
              />
            {:else}
              <img
                src={`https://projectpokemon.org/images/sprites-models/bw-animated/${pokemon.pokedexId
                  .toString()
                  .padStart(3, "0")}.gif`}
                alt={pokemon.name.fr}
                loading="lazy"
                class="sprite"
              />
            {/if}

            <div class="pokemon-card-info">
              <p>N°{pokemon.pokedexId}</p>
              <h2>{pokemon.name.fr}</h2>
              <div class="container type-cont">
                {#if pokemon.types}
                  {#each pokemon.types as type}
                    <div>
                      <img src={type.image} alt={type.name} loading="lazy" />
                      <h4>{type.name.toUpperCase()}</h4>
                    </div>
                  {/each}
                {/if}
              </div>
              {#if pokemon.pokedexId != 0}
                <div class="container capture">
                  <p>Capturé :</p>
                  <div
                    class={caughtPokemon.includes(pokemon.pokedexId)
                      ? "pokeball"
                      : "pokeball notCaught"}
                  >
                    <input
                      type="checkbox"
                      checked={caughtPokemon.includes(pokemon.pokedexId)
                        ? true
                        : false}
                      on:change={() => {
                        if (caughtPokemon.includes(pokemon.pokedexId)) {
                          caughtPokemon = caughtPokemon.map((id) =>
                            id === pokemon.pokedexId ? -1 : id
                          );
                          modifiedPokemon = modifiedPokemon.filter(
                            (pokemon) => pokemon.id !== lastPokemon
                          );
                          localStorage.setItem(
                            "modifiedPokemon",
                            JSON.stringify(modifiedPokemon)
                          );
                        } else {
                          let index = caughtPokemon.findIndex((p) => p === -1);
                          if (index !== -1) {
                            caughtPokemon[index] = pokemon.pokedexId;
                          }
                          caughtPokemon = [...caughtPokemon];

                          modifiedPokemon.push({
                            id: pokemon.pokedexId,
                            name: pokemon.name.fr,
                            shiny: getRandomNumber(1, 4096) == 1 ? true : false,
                          });
                          localStorage.setItem(
                            "modifiedPokemon",
                            JSON.stringify(modifiedPokemon)
                          );
                        }
                        localStorage.setItem(
                          "caughtPokemon",
                          JSON.stringify(caughtPokemon)
                        );
                      }}
                      on:click|stopPropagation
                    />
                  </div>
                </div>
              {/if}
            </div>
          </button>
        {/if}
      {:else if typeFilterOn.indexOf(pokemon.types[0].name) != -1}
        <button
          class="pokemon-card"
          on:click={() => {
            currentPokemon = pokemon;
            pokemonInfoTab("show");
          }}
        >
          {#if pokemon.sprites.animated == undefined && pokemon.pokedexId > 650}
            <img
              src={`https://projectpokemon.org/images/normal-sprite/${pokemon.name.en.toLowerCase()}.gif`}
              alt={pokemon.name.fr}
              loading="lazy"
              class="sprite"
            />
          {:else if pokemon.pokedexId > 649 || pokemon.pokedexId == 0}
            <img
              src={pokemon.sprites.animated}
              alt={pokemon.name.fr}
              loading="lazy"
              class="sprite"
            />
          {:else}
            <img
              src={`https://projectpokemon.org/images/sprites-models/bw-animated/${pokemon.pokedexId
                .toString()
                .padStart(3, "0")}.gif`}
              alt={pokemon.name.fr}
              loading="lazy"
              class="sprite"
            />
          {/if}

          <div class="pokemon-card-info">
            <p>N°{pokemon.pokedexId}</p>
            <h2>{pokemon.name.fr}</h2>
            <div class="container type-cont">
              {#if pokemon.types}
                {#each pokemon.types as type}
                  <div>
                    <img src={type.image} alt={type.name} loading="lazy" />
                    <h4>{type.name.toUpperCase()}</h4>
                  </div>
                {/each}
              {/if}
            </div>
            {#if pokemon.pokedexId != 0}
              <div class="container capture">
                <p>Capturé :</p>
                <div
                  class={caughtPokemon.includes(pokemon.pokedexId)
                    ? "pokeball"
                    : "pokeball notCaught"}
                >
                  <input
                    type="checkbox"
                    checked={caughtPokemon.includes(pokemon.pokedexId)
                      ? true
                      : false}
                    on:change={() => {
                      if (caughtPokemon.includes(pokemon.pokedexId)) {
                        caughtPokemon = caughtPokemon.map((id) =>
                          id === pokemon.pokedexId ? -1 : id
                        );
                        modifiedPokemon = modifiedPokemon.filter(
                          (pokemon) => pokemon.id !== lastPokemon
                        );
                        localStorage.setItem(
                          "modifiedPokemon",
                          JSON.stringify(modifiedPokemon)
                        );
                      } else {
                        let index = caughtPokemon.findIndex((p) => p === -1);
                        if (index !== -1) {
                          caughtPokemon[index] = pokemon.pokedexId;
                        }
                        caughtPokemon = [...caughtPokemon];

                        modifiedPokemon.push({
                          id: pokemon.pokedexId,
                          name: pokemon.name.fr,
                          shiny: getRandomNumber(1, 4096) == 1 ? true : false,
                        });
                        localStorage.setItem(
                          "modifiedPokemon",
                          JSON.stringify(modifiedPokemon)
                        );
                      }
                      localStorage.setItem(
                        "caughtPokemon",
                        JSON.stringify(caughtPokemon)
                      );
                    }}
                    on:click|stopPropagation
                  />
                </div>
              </div>
            {/if}
          </div>
        </button>
      {/if}
    {/if}
  {/each}
</main>

<style>
  @keyframes Shake {
    0%,
    80%,
    100% {
      transform: translate(-50%, -50%) scale(1.3);
    }
    20% {
      transform: translate(-50%, -50%) rotate(15deg) scale(1.3);
    }
    60% {
      transform: translate(-50%, -50%) rotate(-15deg) scale(1.3);
    }
  }

  @keyframes Wave {
    0%, 100% {
      transform: translateY(-5px) scale(1.1);
    }
    50% {
      transform: translateY(5px) scale(1.1);
    }
  }

  @font-face {
    font-family: "Cruinn";
    src: url("/fonts/Cruinn-Regular.ttf");
  }

  * {
    font-family: "Cruinn", sans-serif;
  }

  :global(body) {
    background-color: #f6f8fc;
    margin: 0;
    padding: 0;
  }

  button {
    cursor: pointer;
  }

  button:not(.pokemon-card) {
    font-weight: 600;
    font-size: 16px;
  }

  h2,
  p {
    margin: 0px;
  }

  h3 {
    margin-block: 20px 10px;
  }

  h4 {
    margin: 0;
  }

  .popup {
    position: fixed;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    z-index: 5;
    display: flex;
    justify-content: center;
    align-items: center;
    display: none;
    backdrop-filter: blur(3px);
    z-index: 8;
  }

  .popup > div {
    width: 70%;
    height: 70%;
    background-color: #ffffff;
    border-radius: 10px;
    display: flex;
    border: solid;
    flex-direction: column;
    gap: 30px;
    overflow: scroll;
    /* align-items: center; */
    padding: 5%;
  }

  .popup > div > h2 {
    text-align: center;
  }

  .popup > div li {
    margin-block: 5px;
  }

  #help-content > div {
    display: flex;
    justify-content: space-evenly;
    align-items: center;
    width: 100%;
  }

  #help-content > div > div {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  #boite-pc {
    position: fixed;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    z-index: 5;
    display: flex;
    justify-content: space-evenly;
    align-items: center;
    display: none;
    backdrop-filter: blur(3px);
    gap: 10px;
  }

  #boite-pc > div:first-child {
    /* width: min(90vh, 90vw); */
    /* width: max(min(90vh, 90vw), calc(90vw - 300px)); */
    width: min(90vh, calc(90vw - 250px));
    display: flex;
    gap: 5px;
    flex-direction: column;
  }

  #boite-pc > div:first-child .container {
    justify-content: space-between;
  }

  #storage {
    width: min(90vh, calc(90vw - 250px));
    height: min(90vh, calc(90vw - 250px));
    background-color: #ffffff;
    border: solid;
    border-radius: 10px;
    display: grid;
    /* divier en 30 carré */
    grid-template-columns: repeat(10, 1fr);
    grid-template-rows: repeat(10, 1fr);
    overflow: hidden;
    filter: none !important;
  }

  .storage-case {
    width: 100%;
    height: 100%;
    border: rgb(246, 246, 246) solid 0.1px;
    display: flex;
    justify-content: center;
  }

  .storage-case:hover {
    background-color: #f6f8fc;
  }

  #storage img {
    width: calc(min(90vh, calc(90vw - 250px)) / 10);
    height: calc(min(90vh, calc(90vw - 250px)) / 10);
    object-fit: contain;
    position: absolute;
  }

  img.active {
    animation: Wave 3s ease-in-out infinite;
  }

  img.isDragging {
    animation: Shake 0.8s ease-in-out infinite;
    pointer-events: none;
    z-index: 3;
  }

  .context-menu {
    position: absolute;
    width: 100px;
    height: 100px;
    background-color: #ffffff;
    border-radius: 10px;
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
    align-items: center;
    z-index: 3;
    transform: translateY(-105px);
    border: solid;
    border-radius: 10px;
    overflow: hidden;
  }

  .context-menu button {
    width: 100%;
    height: 33.33%;
    border: none;
    background-color: #ffffff;
    cursor: pointer;
  }

  .context-menu button:hover {
    background-color: #f6f8fc;
  }

  #pokemon-resume {
    background-color: #ffffff;
    margin-top: 45px;
    border-radius: 8px;
    height: calc(min(90vh, 90vw) - 20px);
    flex: 0 0 300px;
    border: solid;
    padding: 10px;
  }

  #pokemon-resume > .container:first-child {
    border-bottom: solid;
    padding-bottom: 5px;
    justify-content: space-between;
  }

  #pokemon-resume h3 {
    display: flex;
  }

  #pokemon-resume h3 input {
    font-size: 26px;
    font-weight: bold;
    width: 100%;
    /* cursor: pointer; */
    padding: 0;
    border: none;
    z-index: 2;
    background: none;
  }

  #pokemon-resume h3 svg {
    position: absolute;
    transform: translateX(270px);
    z-index: 1;
  }

  #pokemon-resume img:not(.type) {
    width: 100%;
    object-fit: cover;
    margin-bottom: 40px;
  }

  /* ============= HEADER ============= */

  .openPC {
    position: fixed;
    border: solid;
    border-radius: 25%;
    padding: 10px;
    border-radius: 100px;
    height: 40px;
    margin: 10px;
    z-index: 3;
  }

  .popup-button-cont {
    position: fixed !important;
    bottom: 10px;
    left: 10px;
    width: 220px !important;
    justify-content: space-between !important;
    z-index: 10 !important;
  }

  .popup-button {
    height: 40px;
    width: 40px;
    border-radius: 100px;
    border: solid;
  }

  :global(.active-popup-button) {
    position: fixed;
    bottom: 10px;
    left: 50%;
    transform: translateX(-50%);
  }

  .credits-button {
    width: fit-content;
  }



  header > div {
    position: fixed;
    margin-inline: 10px;
  }

  .container {
    display: flex;
    justify-content: space-evenly;
    gap: 10px;
    width: 100%;
    flex-wrap: wrap;
    align-items: center;
    z-index: 1;
    position: relative;
  }

  #filter {
    width: 220px;
    margin-block: 60px;
    height: calc(100vh - 130px);
    background-color: #ffffff;
    padding: 15px 15px 0 15px;
    border-radius: 10px;
  }

  #filter .list > div:last-child {
    margin-bottom: 10px;
  }

  #filter > h3 {
    margin-top: 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  #filter > h3 svg {
    display: none;
  }

  #pokemon-info {
    width: 365px;
    right: 0;
    height: 100vh;
    overflow-y: scroll;
    transition: 0.5s;
  }

  #pokemon-info.hide {
    right: -375px;
  }

  #pokemon-info h2,
  #pokemon-info .pokemonId {
    text-align: center;
  }

  .close-button {
    background-color: #ffffff;
    border-radius: 50%;
    border: none;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    width: 40px;
    height: 40px;
  }

  #close-info {
    margin-top: 20px;
  }

  #pokemon-info .info-tab {
    background-color: #ffffff;
    border-radius: 8px;
    margin-top: 160px;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 0 10px 10px 10px;
    margin-bottom: 10px;
    margin-right: 10px;
    gap: 10px;
  }

  #pokemon-info .sprite {
    width: 70%;
    transform: translateY(-70%);
    position: absolute;
  }

  #pokemon-info .pokemonId {
    margin-top: 100px;
  }

  #pokemon-info .talent-cont {
    gap: 0;
  }

  #pokemon-info .talent {
    border: rgb(109, 132, 175) solid;
    background-color: #f6f8fc;
    border-radius: 50px;
    padding: 10px;
    flex: 0 0 130px;
    height: fit-content;
    display: flex;
    justify-content: space-between;
  }

  #pokemon-info .talent-cache {
    border: rgb(181, 12, 12) solid;
  }

  #pokemon-info .single-info {
    display: flex;
    flex-direction: column;
    gap: 0;
    align-items: center;
  }

  #pokemon-info .single-info p {
    background-color: #f6f8fc;
    border-radius: 50px;
    padding: 10px;
    width: 135px;
    text-align: center;
  }

  #pokemon-info .type-list {
    background-color: #f6f8fc;
    border-radius: 50px;
    padding: 10px;
  }

  #pokemon-info .type-list img {
    width: 25px;
    height: 25px;
  }

  #pokemon-info h3 {
    text-align: center;
  }

  #pokemon-info .evolution,
  #pokemon-info .evolution > div {
    flex-direction: column;
    align-items: center;
    gap: 25px;
  }

  #pokemon-info .evolution .image-cont {
    background-color: #f6f8fc;
    border-radius: 50%;
    padding: 10px;
    width: 50px;
    height: 50px;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #pokemon-info .evolution img {
    width: 40px;
    height: 40px;
    object-fit: contain;
  }

  #pokemon-info .evolution .mega-form {
    flex-direction: row;
  }

  #pokemon-info .evolution .mega-form > div {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  #pokemon-info .evolution .mega-form p {
    margin-bottom: 20px;
  }

  #pokemon-info .stats > div {
    flex-direction: column;
  }

  /* ======================= MAIN ========================== */

  main {
    transition: 0.5s;
    margin-inline: 270px 0px;
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
    justify-content: space-evenly;
  }

  :global(main.crop-right) {
    margin-right: 380px !important;
  }

  .pokemon-card {
    width: 250px;
    height: fit-content;
    background-color: #ffffff;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
    margin-top: 50px;
    border: none;
    cursor: pointer;
    min-height: 202px;
  }

  .pokemon-card .sprite {
    transform: translateY(-50%);
    position: absolute;
  }

  .pokemon-card-info {
    margin-top: 50px;
    width: 100%;
    display: flex;
    flex-direction: column;
    gap: 10px;
    padding-bottom: 10px;
    z-index: 2;
    position: relative;
  }

  .pokemon-card-info .container {
    padding-block: 5px 5px;
  }

  .container > div {
    display: flex;
    align-items: center;
    gap: 5px;
  }

  .container > div img {
    width: 25px;
    height: 25px;
  }

  #pokemon-info .container > .talent img {
    width: 20px;
    height: 20px;
  }

  .container.list {
    flex-direction: column;
    gap: 10px;
    align-items: flex-start;
  }

  .container.capture {
    justify-content: center;
  }

  .pokeball {
    display: block;
    width: 25px;
    height: 25px;
    background: radial-gradient(
        white 2px,
        black 2.1px 2.25px,
        white 2.375px 3px,
        black 3.125px 4px,
        transparent 4.125px
      ),
      linear-gradient(
        to bottom,
        red 0 10px,
        black 10.125px 12px,
        white 12.125px 12.5px
      );
    border-radius: 50%;
    border: 1px solid black;
    box-shadow: inset -2px -1px 0 0 rgba(0, 0, 0, 0.2);
    animation:
      fall 0.5s ease-in-out 1s,
      shake 1.25s cubic-bezier(0.36, 0.07, 0.19, 0.97) 1.5s 3,
      catch 0.5s ease-out 5.25s forwards;
  }

  .notCaught {
    filter: grayscale(100%);
  }

  .pokeball input {
    width: 100%;
    height: 100%;
    margin: 0;
    opacity: 0;
    cursor: pointer;
  }

  @media screen and (max-width: 900px) {
    h3 {
      margin-block: 10px;
    }

    .openPC {
      top: 0px;
    }

    #pokemon-info:not(.hide) + .popup-button-cont  {
      bottom: 175px;
    }

    #filter {
      /* display: none; */
      right: 0;
      height: 34px;
      margin-top: 10px;
      top: 0;
      z-index: 4;
      width: fit-content;
      display: flex;
      flex-direction: column;
      border: solid;
      overflow: hidden;
      padding: 10px 10px 0 10px;
      transition: 0.5s;
    }

    #filter.dropdown-open {
      height: 285px;
    }

    #filter.dropdown-open svg {
      transform: rotate(-180deg);
    }

    #filter .container {
      overflow-y: scroll;
      padding-top: 10px;
      flex-wrap: nowrap;
      justify-content: flex-start;
    }

    #filter > h3 svg {
      display: block;
      transition: 0.5s;
    }

    #pokemon-info,
    main {
      margin-inline: 0;
    }

    #pokemon-info {
      /* height: 240px; */
      height: fit-content;
      width: 100%;
      bottom: 0;
      z-index: 3;
    }

    #pokemon-info.hide {
      right: 0px;
      bottom: -230px;
    }

    #pokemon-info h3 {
      text-align: start;
    }

    #close-info {
      margin: 0 0 2px calc(100% - 50px);
      border: solid;
    }

    .info-tab {
      width: fit-content;
      flex-direction: row !important;
      margin-top: 0 !important;
      height: 160px;
      margin-bottom: 0 !important;
      border: solid;
      gap: 50px !important;
    }

    #pokemon-info .pokemonId {
      margin-top: 0;
    }

    #pokemon-info .sprite {
      width: 100px;
      transform: none;
      position: relative;
    }

    #pokemon-info .talent-cont > div {
      width: 160px;
    }

    #pokemon-info .type-list {
      flex-wrap: nowrap;
    }

    #pokemon-info .evolution,
    #pokemon-info .evolution div,
    #pokemon-info .stats {
      flex-direction: row !important;
      flex-wrap: nowrap;
    }

    #pokemon-info .evolution p {
      width: 80px;
    }

    #pokemon-info .mega-form p {
      width: 125px;
      margin-bottom: 0 !important;
    }

    #pokemon-info .str-weak {
      gap: 0;
    }

    main {
      margin-top: 40px;
    }

    :global(main.crop-right) {
      margin-right: 0 !important;
    }

    #boite-pc {
      flex-direction: column;
    }

    #boite-pc > div:first-child {
      width: min(90vw, calc(90vh - 225px));
    }

    #storage {
      width: min(90vw, calc(90vh - 225px));
      height: min(90vw, calc(90vh - 225px));
      /* width: min(90vh, 90vw);
      height: min(90vh, 90vw); */
    }

    #storage img {
      width: calc(min(90vw, calc(90vh - 225px)) / 10);
      height: calc(min(90vw, calc(90vh - 225px)) / 10);
      /* width: calc(min(90vh, 90vw) / 10);
      height: calc(min(90vh, 90vw) / 10); */
    }

    #pokemon-resume {
      margin-top: 0;
      height: max(225px, calc(90vh - 100vw));
      flex: 0 0 auto;
      margin-inline: 5%;
    }

    #pokemon-resume .resume-body {
      height: calc(100% - 50px);
    }

    #pokemon-resume .resume-body > div {
      display: flex;
      height: calc(100% - 50px);
    }

    .resume-body .sprite {
      margin-bottom: 0 !important;
      width: 50% !important;
      object-fit: contain !important;
    }

    .pokemon-card {
      width: 150px;
      height: 240px;
    }

    .type-cont {
      height: 60px;
      flex-direction: column;
    }
  }
</style>
