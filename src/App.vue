<!-- eslint-disable no-unused-vars -->
<!-- eslint-disable no-plusplus -->
<!-- eslint-disable max-len -->
<!-- eslint-disable vuejs-accessibility/label-has-for -->
<!-- eslint-disable vuejs-accessibility/form-control-has-label -->
<template>
  <div class="container">
    <header class="header">
      <img src="@/assets/logo.png" alt="Logo" class="header__logo" />
      <form class="search" @submit.prevent="searchRecipes()">
        <input
          type="text"
          class="search__field"
          placeholder="Search over 1,000,000 recipes..."
          v-model="searchItem"
          required
        />
        <button class="btn search__btn">
          <svg class="search__icon">
            <use xlink:href="@/assets/icons.svg#icon-search"></use>
          </svg>
          <span>Search</span>
        </button>
      </form>

      <nav class="nav">
        <ul class="nav__list">
          <!-- <li class="nav__item">
            <button class="nav__btn nav__btn--add-recipe">
              <svg class="nav__icon">
                <use xlink:href="@/assets/icons.svg#icon-edit"></use>
              </svg>
              <span>Add recipe</span>
            </button>
          </li> -->
          <li class="nav__item">
            <button class="nav__btn nav__btn--bookmarks">
              <svg class="nav__icon">
                <use xlink:href="@/assets/icons.svg#icon-bookmark"></use>
              </svg>
              <span>Bookmarks</span>
            </button>
            <div class="bookmarks">
              <ul class="bookmarks__list">
                <template v-if="bookmarks.length">
                  <li class="preview" v-for="bookmark in bookmarks" :key="bookmark.id">
                    <a
                      class="preview__link"
                      :class="{'preview__link--active': recipe.id === bookmark.id}"
                      @click="renderRecipe(bookmark.id)"
                      :href="`#${bookmark.id}`"
                    >
                      <figure class="preview__fig">
                        <img :src="bookmark.image_url" alt="bookmark.title" />
                      </figure>
                      <div class="preview__data">
                        <h4 class="preview__title">
                          {{ bookmark.title }}
                        </h4>
                        <p class="preview__publisher">{{ bookmark.publisher }}</p>
                        <div class="preview__user-generated">
                          <svg>
                            <use xlink:href="@/assets/icons.svg#icon-user"></use>
                          </svg>
                        </div>
                      </div>
                    </a>
                  </li>
                </template>
                <template v-else>
                  <div class="message">
                    <div>
                      <svg>
                        <use xlink:href="@/assets/icons.svg#icon-smile"></use>
                      </svg>
                    </div>
                    <p>No bookmarks yet. Find a nice recipe and bookmark it :)</p>
                  </div>
                </template>
              </ul>
            </div>
          </li>
        </ul>
      </nav>
    </header>

    <div class="search-results">
      <div class="spinner" v-if="isPending">
        <svg>
          <use xlink:href="@/assets/icons.svg#icon-loader"></use>
        </svg>
      </div>
      <div class="error" v-else-if="error">
        <div>
          <svg>
            <use xlink:href="@/assets/icons.svg#icon-alert-triangle"></use>
          </svg>
        </div>
        <p>No recipes found for your query. Please try again!</p>
      </div>
      <ul class="results" v-else>
        <li class="preview" v-for="allRecipe in allRecipes" :key="allRecipe.id">
          <a class="preview__link" @click="renderRecipe(allRecipe.id)" :href="`#${allRecipe.id}`"  :class="{'preview__link--active': allRecipe.id === recipe.id}">
            <figure class="preview__fig">
              <img :src="allRecipe.image_url" alt="Test" />
            </figure>
            <div class="preview__data">
              <h4 class="preview__title">{{ allRecipe.title }}</h4>
              <p class="preview__publisher">{{ allRecipe.publisher }}</p>
              <!-- <div class="preview__user-generated">
                <svg>
                  <use xlink:href="@/assets/icons.svg#icon-user"></use>
                </svg>
              </div> -->
            </div>
          </a>
        </li>
      </ul>

      <div class="pagination">
        <template v-if="allResults.length">
          <button class="btn--inline pagination__btn--prev" @click="getPreviousPage()" v-if="page !== 1">
            <svg class="search__icon">
              <use  xlink:href="@/assets/icons.svg#icon-arrow-left"></use>
            </svg>
            <span>page {{ page - 1}}</span>
          </button>
          <button
            class="btn--inline pagination__btn--next"
            @click="getNextPage()"
            v-if="page !== numPages"

          >
            <span>page {{page + 1}}</span>
            <svg class="search__icon">
              <use  xlink:href="@/assets/icons.svg#icon-arrow-right"></use>
            </svg>
          </button>
        </template>
      </div>

      <p class="copyright">
        &copy; Copyright by
        <a class="twitter-link" target="_blank" href="https://twitter.com/jonasschmedtman"
          >Jonas Schmedtmann</a
        >. Use for learning or your portfolio. Don't use to teach. Don't claim as your own.
      </p>
    </div>

    <div class="spinner" v-if="pending">
      <svg>
        <use xlink:href="@/assets/icons.svg#icon-loader"></use>
      </svg>
    </div>

    <div class="recipe">
      <template v-if="showMessage">
        <div class="message">
          <div>
            <svg>
              <use xlink:href="@/assets/icons.svg#icon-smile"></use>
            </svg>
          </div>
          <p>Start by searching for a recipe or an ingredient. Have fun!</p>
        </div>
      </template>

      <template v-else>
        <figure class="recipe__fig">
          <img :src="recipe.image_url" :alt="recipe.id" class="recipe__img" />
          <h1 class="recipe__title">
            <span>{{ recipe.title }}</span>
          </h1>
        </figure>

        <div class="recipe__details">
          <div class="recipe__info">
            <svg class="recipe__info-icon">
              <use xlink:href="@/assets/icons.svg#icon-clock"></use>
            </svg>
            <span class="recipe__info-data recipe__info-data--minutes">{{
              recipe.cooking_time
            }}</span>
            <span class="recipe__info-text">minutes</span>
          </div>
          <div class="recipe__info">
            <svg class="recipe__info-icon">
              <use xlink:href="@/assets/icons.svg#icon-users"></use>
            </svg>
            <span class="recipe__info-data recipe__info-data--people">{{ recipe.servings }}</span>
            <span class="recipe__info-text">servings</span>

            <div class="recipe__info-buttons">
              <button class="btn--tiny btn--increase-servings" @click="decreaseServings()">
                <svg>
                  <use xlink:href="@/assets/icons.svg#icon-minus-circle"></use>
                </svg>
              </button>
              <button class="btn--tiny btn--increase-servings" @click="increaseServings()">
                <svg>
                  <use xlink:href="@/assets/icons.svg#icon-plus-circle"></use>
                </svg>
              </button>
            </div>
          </div>

          <div class="recipe__user-generated">
            <svg>
              <use xlink:href="@/assets/icons.svg#icon-user"></use>
            </svg>
          </div>
          <button class="btn--round" @click="handleBookmark()">
            <svg class="" v-if="recipe.bookmarked">
              <use xlink:href="@/assets/icons.svg#icon-bookmark-fill"></use>
            </svg>
            <svg class="" v-else>
              <use xlink:href="@/assets/icons.svg#icon-bookmark"></use>
            </svg>
          </button>
        </div>

        <div class="recipe__ingredients">
          <h2 class="heading--2">Recipe ingredients</h2>
          <ul class="recipe__ingredient-list">
            <li
              class="recipe__ingredient"
              v-for="ingredient in recipe.ingredients"
              :key="ingredient.description"
            >
              <svg class="recipe__icon">
                <use xlink:href="@/assets/icons.svg#icon-check"></use>
              </svg>
              <div class="recipe__quantity">{{ ingredient.quantity }}</div>
              <div class="recipe__description">
                <span class="recipe__unit">{{ ingredient.unit }}</span>
                {{ ingredient.description }}
              </div>
            </li>
          </ul>
        </div>

        <div class="recipe__directions">
          <h2 class="heading--2">How to cook it</h2>
          <p class="recipe__directions-text">
            This recipe was carefully designed and tested by
            <span class="recipe__publisher">{{ recipe.publisher }}</span
            >. Please check out directions at their website.
          </p>
          <a class="btn--small recipe__btn" :href="recipe.source_url" target="_blank">
            <span>Directions</span>
            <svg class="search__icon">
              <use xlink:href="@/assets/icons.svg#icon-arrow-right"></use>
            </svg>
          </a>
        </div>
      </template>
    </div>
  </div>

  <div class="overlay hidden"></div>
  <div class="add-recipe-window hidden">
    <button class="btn--close-modal">&times;</button>
    <form class="upload">
      <div class="upload__column">
        <h3 class="upload__heading">Recipe data</h3>
        <label>Title</label>
        <input value="TEST" required name="title" type="text" />
        <label>URL</label>
        <input value="TEST" required name="sourceUrl" type="text" />
        <label>Image URL</label>
        <input value="TEST" required name="image" type="text" />
        <label>Publisher</label>
        <input value="TEST" required name="publisher" type="text" />
        <label>Prep time</label>
        <input value="23" required name="cookingTime" type="number" />
        <label>Servings</label>
        <input value="23" required name="servings" type="number" />
      </div>

      <div class="upload__column">
        <h3 class="upload__heading">Ingredients</h3>
        <label>Ingredient 1</label>
        <input
          value="0.5,kg,Rice"
          type="text"
          required
          name="ingredient-1"
          placeholder="Format: 'Quantity,Unit,Description'"
        />
        <label>Ingredient 2</label>
        <input
          value="1,,Avocado"
          type="text"
          name="ingredient-2"
          placeholder="Format: 'Quantity,Unit,Description'"
        />
        <label>Ingredient 3</label>
        <input
          value=",,salt"
          type="text"
          name="ingredient-3"
          placeholder="Format: 'Quantity,Unit,Description'"
        />
        <label>Ingredient 4</label>
        <input type="text" name="ingredient-4" placeholder="Format: 'Quantity,Unit,Description'" />
        <label>Ingredient 5</label>
        <input type="text" name="ingredient-5" placeholder="Format: 'Quantity,Unit,Description'" />
        <label>Ingredient 6</label>
        <input type="text" name="ingredient-6" placeholder="Format: 'Quantity,Unit,Description'" />
      </div>

      <button class="btn upload__btn">
        <svg>
          <use xlink:href="@/assets/icons.svg#icon-upload-cloud"></use>
        </svg>
        <span>Upload</span>
      </button>
    </form>
  </div>
  <router-view />
</template>

<script>
import axios from 'axios';
// import fractional from 'fractional';

export default {
  name: 'App',

  data() {
    return {
      allRecipes: [],
      recipe: [],
      allResults: [],
      start: null,
      end: null,
      resultsPerPage: 10,
      page: 1,
      numPages: null,
      lastPage: null,
      pending: null,
      isPending: false,
      searchItem: '',
      error: false,
      bookmarks: [],
      showMessage: true,
    };
  },

  // Display recipe deta ils even after when browser is refreshed.

  async created() {
    try {
      this.showMessage = true;
      const id = window.location.hash.slice(1);
      console.log(id);
      this.pending = true;
      const res = await axios.get(`https://forkify-api.herokuapp.com/api/v2/recipes/${id}`);
      const { recipe } = res.data.data;
      this.recipe = recipe;
      this.showMessage = false;

      const storage = localStorage.getItem('bookmarks');
      if (storage) {
        this.bookmarks = JSON.parse(storage);
      }

      if (this.bookmarks.some((bookmark) => bookmark.id === id)) {
        this.recipe.bookmarked = true;
      } else {
        this.recipe.bookmarked = false;
      }

      this.pending = true;
    } catch (err) {
      // get bookmarks from local storage
      const storage = localStorage.getItem('bookmarks');
      if (storage) this.bookmarks = JSON.parse(storage);
      console.log(this.bookmarks);
      this.showMessage = true;
    }
    this.pending = false;
  },

  methods: {

    // Query for recipes
    async searchRecipes() {
      try {
        this.isPending = true;
        const res = await axios.get(
          `https://forkify-api.herokuapp.com/api/v2/recipes?search=${this.searchItem}`,
        );
        const { recipes } = res.data.data;

        // store all recipe from the api
        this.allRecipes = recipes;

        // store all the recipes from the result here.
        this.allResults = this.allRecipes;

        // Display only ten recipes
        this.allRecipes = this.getSearchResultsPage(1);
        this.page = 1;

        this.error = false;
        if (this.allRecipes.length === 0 || !this.allRecipes) {
          this.error = true;
        }
      } catch (err) {
        console.log(err);
      }
      this.searchItem = '';
      this.isPending = false;
    },

    // Dispaly recipe details
    async renderRecipe(id) {
      try {
        this.pending = true;
        this.showMessage = false;

        const res = await axios.get(`https://forkify-api.herokuapp.com/api/v2/recipes/${id}`);
        const { recipe } = res.data.data;
        this.recipe = recipe;

        if (this.bookmarks.some((bookmark) => bookmark.id === id)) {
          this.recipe.bookmarked = true;
        } else {
          this.recipe.bookmarked = false;
        }
      } catch (err) {
        console.log(err);
      }
      this.pending = false;
    },

    getSearchResultsPage(page = this.page) {
      this.page = page;
      const results = this.allResults;

      this.start = (page - 1) * this.resultsPerPage; // 0
      this.end = page * this.resultsPerPage; // 9
      return results.slice(this.start, this.end);
    },

    getNextPage() {
      const curPage = this.page;

      this.numPages = Math.ceil(this.allResults.length / this.resultsPerPage);

      // Page 1 and there are other pages
      if (curPage === 1 && this.numPages > 1) {
        this.allRecipes = this.getSearchResultsPage(curPage + 1);
      }

      if (curPage < this.numPages) {
        this.allRecipes = this.getSearchResultsPage(curPage + 1);
      }

      return '';
    },

    getPreviousPage() {
      const curPage = this.page;
      this.numPages = Math.ceil(this.allResults.length / this.resultsPerPage);

      if (curPage < this.numPages && curPage !== 1) {
        this.allRecipes = this.getSearchResultsPage(curPage - 1);
      }

      if (curPage === this.numPages && this.numPages > 1) {
        this.allRecipes = this.getSearchResultsPage(curPage - 1);
      }

      return '';
    },

    // Decrese Servings
    decreaseServings() {
      // eslint-disable-next-line no-plusplus
      if (this.recipe.servings !== 1) {
        // eslint-disable-next-line no-plusplus
        const oldServings = this.recipe.servings--;

        const newServings = this.recipe.servings;

        this.recipe.ingredients.forEach((ing) => {
          if (ing.quantity === null) {
            return ing.quantity;
          }

          // eslint-disable-next-line no-param-reassign
          ing.quantity = (ing.quantity * newServings) / oldServings;
        });
      }
    },

    // increasing servings
    increaseServings() {
      // eslint-disable-next-line no-plusplus, no-unused-vars
      const oldServings = this.recipe.servings++;
      const newServings = this.recipe.servings;
      this.recipe.ingredients.forEach((ing) => {
        if (ing.quantity === null) {
          return ing.quantity;
        }
        // eslint-disable-next-line no-param-reassign
        ing.quantity = (ing.quantity * newServings) / oldServings;
      });
    },

    // SET BOOKMARK
    persistBookmarks() {
      localStorage.setItem('bookmarks', JSON.stringify(this.bookmarks));
    },

    addBookmark(recipe) {
      // Add Bookmark
      this.bookmarks.push(recipe);

      // Mark current recipe as bookmarked
      if (recipe.id === this.recipe.id) this.recipe.bookmarked = true;

      this.persistBookmarks();
    },

    deleteBookMark(id) {
      // Delete Bookmark
      const index = this.bookmarks.findIndex((bookmark) => bookmark.id === id);
      this.bookmarks.splice(index, 1);

      // remove bookmarked properties.
      if (id === this.recipe.id) this.recipe.bookmarked = false;
      this.persistBookmarks();
    },

    handleBookmark() {
      if (this.recipe.bookmarked) {
        this.deleteBookMark(this.recipe.id);
      } else {
        this.addBookmark(this.recipe);
      }
    },
  },
};
</script>

<style lang="scss"></style>
