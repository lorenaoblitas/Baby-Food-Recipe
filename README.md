# Baby-Food-Recipe
Baby Food Recipe Generator 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Real Foods for Babies</title>
    
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom Calm Green & Lavender Theme */
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
            color: #4c4c4c;
            min-height: 100vh;
            
            /* Solid Calm Green Background */
            background-color: #e6ffe6; /* A very soft, calming green */
        }
        
        .recipe-card-details {
            transition: max-height 0.3s ease-out;
            max-height: 0;
            overflow: hidden;
        }
        .recipe-card.expanded .recipe-card-details {
            max-height: 500px;
        }
        .chevron {
            transition: transform 0.3s ease;
        }
        .recipe-card.expanded .chevron {
            transform: rotate(180deg);
        }
        .selection-btn {
            /* Kept soft lavender border and purple text */
            @apply text-lg font-semibold px-6 py-3 rounded-xl bg-white border-2 border-purple-300 text-purple-700 hover:bg-purple-100 transition-all shadow-lg;
        }
        .recipe-filter-btn {
            @apply !text-base !py-2 !px-4 rounded-xl shadow-md;
        }
        /* Styling for active filter (Calming Lavender/Purple) */
        .filter-btn.active {
            background-color: #8b5cf6 !important; /* purple-500 */
            color: white !important;
            font-weight: 600 !important;
            border-color: #8b5cf6 !important;
        }
        /* Custom styling for LLM output text */
        #modal-content p {
            margin-bottom: 1rem;
            line-height: 1.6;
        }
        #modal-content h3 {
            font-weight: 600;
            font-size: 1.125rem;
            /* Softer, calming purple for subheadings */
            color: #581c87; 
            margin-top: 1rem;
            margin-bottom: 0.5rem;
        }
    </style>
</head>
<body class="text-stone-800 min-h-screen">

    
<div class="container mx-auto p-4 max-w-6xl">
        
        <header class="text-center my-8">
            <h1 class="text-4xl font-bold text-teal-700">Real Foods for Babies</h1>
            <p class="text-lg text-stone-600 mt-2">An interactive guide to the 26 essential recipes.</p>
        </header>

        <main class="min-h-[50vh] flex flex-col items-center justify-center p-4">
            
            <div id="age-selector-ui" class="text-center space-y-8 max-w-xl">
                <h2 class="text-2xl font-bold text-teal-700">First, how old is your infant?</h2>
                <div id="age-filter-container" class="flex flex-wrap justify-center gap-4">
                    <button class="selection-btn" data-age="4-6" onclick="handleAgeSelection('4-6')">
                        4-6 Months
                    </button>
                    <button class="selection-btn" data-age="7-9" onclick="handleAgeSelection('7-9')">
                        7-9 Months
                    </button>
                    <button class="selection-btn" data-age="10-12" onclick="handleAgeSelection('10-12')">
                        10-12 Months
                    </button>
                    <button class="selection-btn" data-age="12+" onclick="handleAgeSelection('12+')">
                        12+ Months
                    </button>
                </div>
            </div>

            <div id="meal-selector-ui" class="hidden text-center space-y-8 max-w-xl">
                <h2 class="text-2xl font-bold text-teal-700">Great! What meal are you planning?</h2>
                <div id="meal-filter-container" class="flex flex-wrap justify-center gap-4">
                    <button class="selection-btn" data-meal="All" onclick="handleMealSelection('All')">
                        All Meals
                    </button>
                    <button class="selection-btn" data-meal="Breakfast" onclick="handleMealSelection('Breakfast')">
                        ☀️ Breakfast
                    </button>
                    <button class="selection-btn" data-meal="Lunch" onclick="handleMealSelection('Lunch')">
                        🥗 Lunch
                    </button>
                    <button class="selection-btn" data-meal="Dinner" onclick="handleMealSelection('Dinner')">
                        🍽️ Dinner
                    </button>
                </div>
            </div>
            
            <div id="full-app-view" class="hidden w-full">
                <div class="flex justify-between items-center bg-white/70 backdrop-blur-sm p-4 rounded-xl mb-6 shadow-md border border-purple-100">
                    <p class="text-lg font-semibold text-teal-700">
                        Showing recipes for <span id="current-age-display" class="text-purple-600"></span> 
                        (<span id="current-meal-display" class="text-purple-600"></span>)
                    </p>
                    <button onclick="startOver()" class="text-sm font-medium py-2 px-4 rounded-full bg-red-100 text-red-600 hover:bg-red-200 transition shadow">
                        Start Over
                    </button>
                </div>

                <div id="grid-meal-filter-container" class="flex flex-wrap justify-center gap-3 mb-6">
                    <button class="recipe-filter-btn selection-btn" data-meal="All" onclick="handleMealSelection('All')">
                        All Meals
                    </button>
                    <button class="recipe-filter-btn selection-btn" data-meal="Breakfast" onclick="handleMealSelection('Breakfast')">
                        ☀️ Breakfast
                    </button>
                    <button class="recipe-filter-btn selection-btn" data-meal="Lunch" onclick="handleMealSelection('Lunch')">
                        🥗 Lunch
                    </button>
                    <button class="recipe-filter-btn selection-btn" data-meal="Dinner" onclick="handleMealSelection('Dinner')">
                        🍽️ Dinner
                    </button>
                </div>

                <div id="recipe-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                    

</div>
            </div>
        </main>

    </div>
    

<div id="gemini-modal" class="hidden fixed inset-0 bg-black bg-opacity-40 z-50 flex items-center justify-center p-4" onclick="closeModal()">
        <div class="bg-white rounded-xl shadow-2xl w-full max-w-xl max-h-[90vh] overflow-y-auto" onclick="event.stopPropagation()">
            <div class="p-6">
                <div class="flex justify-between items-start mb-4 border-b pb-3">
                    <h2 id="modal-title" class="text-xl font-bold text-teal-700"></h2>
                    <button class="text-stone-500 hover:text-stone-700 text-2xl" onclick="closeModal()">
                        &times;
                    </button>
                </div>
                <div id="modal-content" class="text-stone-700 text-base">
                    

</div>
                <div id="modal-citations" class="text-xs text-stone-500 mt-4 pt-4 border-t border-stone-100 hidden">
                    

</div>
            </div>
        </div>
    </div>
    
    <script>
        const GEMINI_API_KEY = "";
        const GEMINI_API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=" + GEMINI_API_KEY;

        const APP_STATES = {
            AGE_SELECTION: 'AGE_SELECTION',
            MEAL_SELECTION: 'MEAL_SELECTION',
            RECIPE_GRID: 'RECIPE_GRID'
        };

        // Global state for filtering
        let currentState = APP_STATES.AGE_SELECTION;
        let currentAgeFilter = null;
        let currentMealFilter = null;
        
        // Map defining which age categories are included when a specific age is selected (CUMULATIVE FILTERING)
        const includedAgesMap = {
            "4-6": ["4-6"],
            "7-9": ["4-6", "7-9"],
            "10-12": ["4-6", "7-9", "10-12"],
            // 12+ now includes all previous groups (4-6, 7-9, 10-12)
            "12+": ["4-6", "7-9", "10-12", "12+"] 
        };

        const allRecipes = [
            // 4-6 Months
            // Note: For 4-9M, most recipes are suitable for both lunch/dinner, so they will be tagged "Lunch" or "Dinner".
            { id: 1, icon: "🍠", name: "Sweet Potato Puree", age: "4-6", meal: "Lunch", ingredients: "1 medium sweet potato.", notes: "Steam or roast until very soft. Blend with a few tablespoons of breast milk, formula, or water until completely smooth." },
            { id: 2, icon: "🥑", name: "Avocado Puree", age: "4-6", meal: "Lunch", ingredients: "1 ripe avocado.", notes: "Mash thoroughly with a fork, or blend with a splash of water/milk. Serve immediately to avoid browning. High in healthy fats." },
            { id: 3, icon: "🟢", name: "Green Pea Puree", age: "4-6", meal: "Dinner", ingredients: "1 cup frozen green peas.", notes: "Steam until tender. Blend with liquid until silky smooth. Sieve if necessary to remove small husks." },
            { id: 4, icon: "🥣", name: "Oatmeal Puree", age: "4-6", meal: "Breakfast", ingredients: "2 tbsp rolled oats or iron-fortified baby oatmeal, liquid.", notes: "If using rolled oats, cook thoroughly and blend until completely smooth. If using baby cereal, prepare according to package directions, thinning generously with breast milk, formula, or water to a very thin consistency." },
            { id: 5, icon: "🍐", name: "Baked Pear Puree", age: "4-6", meal: "Lunch", ingredients: "1 ripe pear (Bosc or Anjou).", notes: "Peel, core, and bake until soft. Blend until smooth. Pears are often easier to digest than apples." },
            { id: 24, icon: "🥑🍌", name: "Avocado & Banana Mash", age: "4-6", meal: "Breakfast", ingredients: "1/2 ripe avocado, 1/2 ripe banana.", notes: "Mash or blend thoroughly until smooth. Banana provides sweetness and texture, while avocado provides healthy fats. Serve immediately." },
            { id: 25, icon: "🍍🍐", name: "Pineapple & Pear Blend", age: "4-6", meal: "Dinner", ingredients: "1/2 cup cooked pear (peeled), 1/4 cup fresh or cooked pineapple.", notes: "Steam or bake the pear until soft. Cook the pineapple briefly (optional, to reduce acidity). Blend together until very smooth. A great source of Vitamin C." },

            
            // 7-9 Months
            { id: 6, icon: "🥕🍊", name: "Carrot Orange Ginger", age: "7-9", meal: "Lunch", ingredients: "1 cup carrots, 1/4 cup freshly squeezed orange juice, tiny slice of fresh ginger.", notes: "Steam carrots until very soft. Blend with the orange juice and a *tiny* piece of ginger (no more than the size of a fingernail) to reach desired puree consistency. Orange adds Vitamin C and bright flavor." },
            { id: 7, icon: "🐔🥑", name: "Lentil Avocado Chicken Mash", age: "7-9", meal: "Dinner", ingredients: "1/4 cup cooked lentils, 1/4 ripe avocado, 1/4 cup finely shredded chicken.", notes: "Mash the avocado and shredded chicken together with the cooked lentils. A great source of protein, iron, and healthy fats. Mash well for a smooth but slightly chunky texture." },
            { id: 23, icon: "🥕🍎🍠", name: "Carrot, Apple & Sweet Potato Blend", age: "7-9", meal: "Lunch", ingredients: "1/2 cup chopped carrots, 1/2 chopped apple (peeled), 1/2 cup sweet potato (chopred).", notes: "Steam all ingredients until fork-tender. Blend until smooth or slightly chunky, adjusting with water/milk as needed." },
            { id: 9, icon: "🥜", name: "Banana & Almond Butter Blend", age: "7-9", meal: "Breakfast", ingredients: "1 ripe banana, 1 tsp smooth, unsalted almond butter.", notes: "Mash banana well. Stir in almond butter. This is a common method for early introduction of allergens. **Ensure nut butter is very smooth and thinned with liquid/fruit to prevent choking.** (Check with pediatrician first)." },
            { id: 10, icon: "🍑", name: "Quinoa & Peach Mash", age: "7-9", meal: "Dinner", ingredients: "1/4 cup cooked quinoa, 1 ripe peach (steamed/baked).", notes: "Combine the cooked quinoa with mashed peaches, leaving some small quinoa texture for practice." },
            { id: 22, icon: "💖", name: "Beet & Strawberry Blend", age: "7-9", meal: "Lunch", ingredients: "1 medium cooked beet, 1/2 cup fresh strawberries, optional: 1/4 cup plain yogurt.", notes: "Steam or roast the beet until very tender. Combine the cooked beet and washed strawberries in a blender. Add yogurt or water to reach a smooth, spoonable consistency. Great for iron and Vitamin C." },
            { id: 26, icon: "🥚", name: "Soft Boiled Egg Mash", age: "7-9", meal: "Breakfast", ingredients: "1 hard-boiled egg (yolk and white).", notes: "Chop or mash the entire egg thoroughly until it reaches a safe, slightly lumpy consistency. Mix with breast milk, formula, or water if needed. An excellent source of iron and choline. (Ensure thorough cooking)." },
            { id: 27, icon: "🐔🍠", name: "Sweet Potato & Chicken Mash", age: "7-9", meal: "Dinner", ingredients: "1/2 cup cooked sweet potato, 1/4 cup finely shredded cooked chicken.", notes: "Mash or blend the steamed sweet potato and finely shredded chicken together. Use a little liquid to reach a soft, mashable consistency. Excellent source of Vitamin A and protein." },
            { id: 28, icon: "🥑🟢🐔", name: "Green Bean, Avocado & Chicken Mash", age: "7-9", meal: "Lunch", ingredients: "1/2 cup steamed green beans, 1/4 ripe avocado, 1/4 cup finely shredded cooked chicken.", notes: "Steam green beans until very soft. Mash or blend all ingredients together. Avocados provide healthy fats for brain development." },
            { id: 29, icon: "🥩🥑🟢", name: "Green Bean, Avocado & Beef", age: "7-9", meal: "Dinner", ingredients: "1/2 cup steamed green beans, 1/4 ripe avocado, 1/4 cup finely ground cooked beef.", notes: "Steam green beans until very soft. Mash or blend all ingredients together. Red meat is an excellent source of crucial heme iron." },
            { id: 30, icon: "🥩🍠", name: "Sweet Potato & Beef Mash", age: "7-9", meal: "Dinner", ingredients: "1/2 cup cooked sweet potato, 1/4 cup finely ground cooked beef.", notes: "Mash or blend the steamed sweet potato and ground beef together until a smooth consistency is reached. A power combination of iron and Vitamin A." },
            { id: 32, icon: "🐟🍠", name: "Sweet Potato & Salmon Mash", age: "7-9", meal: "Dinner", ingredients: "1/2 cup cooked sweet potato, 1/4 cup flaked cooked salmon.", notes: "Mash the steamed sweet potato and flaked salmon together. Salmon is rich in Omega-3s. Ensure all bones are removed. Use a safe, mild fish like salmon or cod." },
            { 
                id: 33, 
                icon: "🍓🍌🫐", 
                name: "Strawberry Banana Blueberry Puree", 
                age: "7-9", 
                meal: "Breakfast", 
                ingredients: "1/2 cup fresh strawberries, 1/2 ripe banana, 1/4 cup blueberries.", 
                notes: "Wash and prepare fruits (hull strawberries, peel banana). Blend all ingredients until smooth. Add a splash of water or breast milk/formula if needed for desired consistency. A vibrant and nutritious blend.",
                // Using the latest available file reference for the uploaded image
                image: "uploaded:Recipes for Baby.jpg-692d5ef3-b6d2-4d6e-b03b-085b1d60c194"
            },
            
            // 10-12 Months
            { id: 11, icon: "🐟", name: "Mini Salmon Cakes", age: "10-12", meal: "Dinner", ingredients: "Canned/cooked salmon, mashed sweet potato, *optional binder* (e.g., fine oatmeal).", notes: "Mix and shape into small, flat patties. Bake or lightly pan-fry until cooked through. Soft and easily held. Use a small amount of fine oatmeal or cooked rice to help the patties hold shape." },
            { id: 12, icon: "🍳", name: "Scrambled Egg with Spinach", age: "10-12", meal: "Breakfast", ingredients: "1 egg, handful of finely chopped cooked spinach.", notes: "Scramble well in a small amount of oil or butter. Chop into small, manageable pieces." },
            { id: 13, icon: "⚫", name: "Black Bean Sweet Potato Cubes", age: "10-12", meal: "Lunch", ingredients: "Cooked black beans (mashed), sweet potato (baked, cubed).", notes: "Serve the baked sweet potato in cubes (finger-sized sticks) alongside mashed black beans for dipping or scooping." },
            { id: 15, icon: "🍚", name: "Chicken and Rice Porridge", age: "10-12", meal: "Dinner", ingredients: "Finely shredded chicken breast, well-cooked rice.", notes: "Mix finely shredded chicken and well-cooked rice. Use water or breast milk/formula to thin the mix if needed, creating a thick, lumpy porridge." },
            { 
                id: 31, 
                icon: "🥣🍓🫐", 
                name: "Quinoa & Berry Breakfast Bowl", 
                age: "10-12", 
                meal: "Breakfast", 
                ingredients: "1/4 cup cooked quinoa, 1/2 banana (mashed), small handful of blueberries (quartered), small handful of strawberries (diced).", 
                notes: "Combine all ingredients. Quinoa provides protein and iron, while berries add Vitamin C. Ensure all berries are quartered or mashed to prevent choking."
            },
            
            // 12+ Months
            { id: 17, icon: "🌶️", name: "Mild Turkey Chili", age: "12+", meal: "Dinner", ingredients: "Ground turkey, cooked lentils, kidney beans, mild spices (cumin).", notes: "Cook thoroughly and mash larger beans if needed. Serve warm as a hearty spoon-fed meal." },
            { id: 18, icon: "🥚", name: "Breakfast Egg Muffins", age: "12+", meal: "Breakfast", ingredients: "Eggs, grated zucchini, mild herbs (optional).", notes: "Whisk eggs and grated zucchini (and herbs, if using). Pour into a muffin tin and bake until set. Cut into strips or cubes for easy self-feeding." },
            { id: 21, icon: "💜", name: "Pink Sultan", age: "12+", meal: "Lunch", ingredients: "1 roasted or steamed beet, 1 cup plain whole-milk yogurt, fresh basil leaves.", notes: "Roast or steam the beet until very tender, then peel. Blend the beet and basil leaves until smooth. Fold the beet mixture into the yogurt. This is a cold, refreshing dip or side. **Note:** Omit salt/seasoning for children under 2." }
        ];

        const grid = document.getElementById('recipe-grid');
        const modal = document.getElementById('gemini-modal');
        const modalTitle = document.getElementById('modal-title');
        const modalContent = document.getElementById('modal-content');
        const modalCitations = document.getElementById('modal-citations');

        // --- Core Application Functions for State Management and Filtering ---

        /**
         * Updates the UI state to show the correct screen (Age, Meal, or Grid).
         */
        function updateUI() {
            // Hide all primary content panels
            document.getElementById('age-selector-ui').classList.add('hidden');
            document.getElementById('meal-selector-ui').classList.add('hidden');
            document.getElementById('full-app-view').classList.add('hidden');
            
            // Show the correct panel based on state
            if (currentState === APP_STATES.AGE_SELECTION) {
                document.getElementById('age-selector-ui').classList.remove('hidden');
            } else if (currentState === APP_STATES.MEAL_SELECTION) {
                document.getElementById('meal-selector-ui').classList.remove('hidden');
            } else if (currentState === APP_STATES.RECIPE_GRID) {
                // Show the recipe grid view
                document.getElementById('full-app-view').classList.remove('hidden');
                // Apply filters and update display text
                applyFilters(); 
                updateDisplayInfo();
            }
        }

        /**
         * Updates the display text in the final view to show active filters and highlights the active button.
         */
        function updateDisplayInfo() {
            let ageText = currentAgeFilter + ' Months';
            if (currentAgeFilter === '12+') {
                 // Clarify that 12+ includes all previous stages
                 ageText = '12+ Months (All Stages Included)'; 
            } else if (currentAgeFilter === '10-12') {
                ageText = '10-12 Months (and Younger Stages)';
            } else if (currentAgeFilter === '7-9') {
                ageText = '7-9 Months (and Younger Stages)';
            }
            document.getElementById('current-age-display').textContent = ageText;


            let mealText = currentMealFilter === 'All' ? 'All Meals' : 
                           currentMealFilter === 'Breakfast' ? 'Breakfast' : 
                           currentMealFilter === 'Lunch' ? 'Lunch' : 'Dinner';
            document.getElementById('current-meal-display').textContent = mealText;

            // Highlight the correct filter button in the grid view
            updateMealFilterButtons();
        }

        /**
         * Highlights the active meal filter button in the grid view.
         */
        function updateMealFilterButtons() {
            // Target the buttons in the grid's filter bar
            document.querySelectorAll('.recipe-filter-btn').forEach(btn => {
                btn.classList.remove('filter-btn', 'active');
            });

            // Add active class to the current button
            const activeBtn = document.querySelector(`.recipe-filter-btn[data-meal="${currentMealFilter}"]`);
            if (activeBtn) {
                activeBtn.classList.add('filter-btn', 'active');
            }
        }


        /**
         * Renders the recipe cards and filters them based on currentAgeFilter and currentMealFilter (cumulative).
         */
        function applyFilters() {
            grid.innerHTML = ''; // Clear previous content

            // Get all age categories that should be included based on currentAgeFilter
            const allowedAges = includedAgesMap[currentAgeFilter] || [];

            const filteredRecipes = allRecipes.filter(recipe => {
                // Check if the recipe's age tag is in the list of allowed ages for the selected category
                const ageMatch = allowedAges.includes(recipe.age);
                
                // Check if the meal matches the filter (or if the filter is "All")
                const mealMatch = currentMealFilter === 'All' || recipe.meal === currentMealFilter;
                
                return ageMatch && mealMatch;
            });

            if (filteredRecipes.length > 0) {
                filteredRecipes.forEach(createRecipeCard);
            } else {
                grid.innerHTML = '<p class="text-center text-lg text-stone-500 col-span-full py-12">No recipes found for this combination. Try selecting "All Meals" or choose a younger age group!</p>';
            }
        }

        function handleAgeSelection(age) {
            currentAgeFilter = age;
            currentState = APP_STATES.MEAL_SELECTION;
            updateUI();
        }

        function handleMealSelection(meal) {
            currentMealFilter = meal;
            
            // If currently in the MEAL_SELECTION step, transition to RECIPE_GRID
            if (currentState === APP_STATES.MEAL_SELECTION) {
                currentState = APP_STATES.RECIPE_GRID;
            }
            
            // If already in RECIPE_GRID, this call updates the filter and re-renders the grid
            updateUI();
        }

        function startOver() {
            currentState = APP_STATES.AGE_SELECTION;
            currentAgeFilter = null;
            currentMealFilter = null;
            grid.innerHTML = ''; // Clear grid
            updateUI();
        }

        function createRecipeCard(recipe) {
            const card = document.createElement('div');
            card.className = 'recipe-card bg-white rounded-lg shadow-md overflow-hidden border border-stone-200 transition-all hover:shadow-lg';
            card.dataset.age = recipe.age;
            card.dataset.meal = recipe.meal; 
            card.dataset.id = recipe.id;

            // Generate image HTML if an image path is provided
            // We use the file ID directly for the uploaded image.
            const imageHtml = recipe.image ? 
                `<img src="${recipe.image}" alt="${recipe.name}" class="w-full h-40 object-cover rounded-md mb-3" onerror="this.onerror=null; this.src='https://placehold.co/600x400/cccccc/000000?text=No+Image';">` : 
                '';
            
            card.innerHTML = `
                <div class="p-4 cursor-pointer" onclick="toggleDetails(${recipe.id})">
                    <div class="flex justify-between items-center">
                        <h3 class="text-lg font-semibold text-teal-700">${recipe.icon} ${recipe.name}</h3>
                        <span class="chevron text-xl text-stone-400">▼</span>
                    </div>
                </div>
                <div id="details-${recipe.id}" class="recipe-card-details px-4 pb-4">
                    <div class="border-t border-stone-100 pt-3">
                        ${imageHtml}
                        <h4 class="font-semibold text-stone-700 mb-1">Ingredients:</h4>
                        <p class="text-sm text-stone-600 mb-3">${recipe.ingredients}</p>
                        <h4 class="font-semibold text-stone-700 mb-1">Preparation:</h4>
                        <p class="text-sm text-stone-600">${recipe.notes}</p>
                        
                        <div class="flex space-x-2 mt-4">
                            <button onclick="runCustomizer(${recipe.id}); event.stopPropagation();" class="text-xs font-semibold py-1 px-2 rounded-full bg-purple-100 text-purple-700 hover:bg-purple-200 transition flex items-center shadow">
                                ✨ Customize Recipe
                            </button>
                            <button onclick="runAnalysis(${recipe.id}); event.stopPropagation();" class="text-xs font-semibold py-1 px-2 rounded-full bg-amber-100 text-amber-700 hover:bg-amber-200 transition flex items-center shadow">
                                🧠 Quick Nutritional Check
                            </button>
                        </div>
                        
                    </div>
                </div>
            `;
            grid.appendChild(card);
        }

        function toggleDetails(id) {
            const card = grid.querySelector(`[data-id="${id}"]`);
            card.classList.toggle('expanded');
        }
        
        // --- Modal and Gemini Functions ---

        function showModal(title, content) {
            modalTitle.textContent = title;
            modalContent.innerHTML = content;
            modalCitations.classList.add('hidden');
            modalCitations.innerHTML = '';
            modal.classList.remove('hidden');
        }

        function showLoading(title) {
            showModal(title, `
                <div class="flex items-center justify-center p-8">
                    <svg class="animate-spin h-6 w-6 text-purple-600 mr-3" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <p>Thinking like a pediatric chef...</p>
                </div>
            `);
        }
        
        function closeModal() {
            modal.classList.add('hidden');
        }


        const retryFetch = async (url, options, maxRetries = 5) => {
            for (let i = 0; i < maxRetries; i++) {
                try {
                    const response = await fetch(url, options);
                    if (response.ok) return response;
                    if (response.status === 429 && i < maxRetries - 1) { 
                        const delay = Math.pow(2, i) * 1000 + Math.random() * 1000;
                        await new Promise(resolve => setTimeout(resolve, delay));
                        continue;
                    }
                    throw new Error(`HTTP error! status: ${response.status}`);
                } catch (error) {
                    if (i === maxRetries - 1) throw error;
                }
            }
        };

        async function runCustomizer(recipeId) {
            const recipe = allRecipes.find(r => r.id === recipeId);
            if (!recipe) return;

            showLoading(`Customizing "${recipe.name}"`);

            const userQuery = `You are a creative baby food chef. Take the following recipe for a ${recipe.age} month old: Recipe Name: ${recipe.name}, Ingredients: ${recipe.ingredients}, Notes: ${recipe.notes}. Suggest one major substitution (e.g., swapping protein or fruit) AND suggest a different preparation method suitable for the same age range. Provide the new recipe details in clear, concise paragraphs with titles using H3 markdown.`;
            
            const systemPrompt = "You are an imaginative and safe baby food recipe modifier. Provide constructive substitutions and alternative cooking methods that maintain the nutritional value and consistency appropriate for the baby's age. Format your response in clean Markdown paragraphs and use H3 headers for sections.";

            try {
                const payload = {
                    contents: [{ parts: [{ text: userQuery }] }],
                    systemInstruction: { parts: [{ text: systemPrompt }] },
                };

                const options = {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                };

                const response = await retryFetch(GEMINI_API_URL, options);
                const result = await response.json();
                const text = result.candidates?.[0]?.content?.parts?.[0]?.text || "Sorry, I couldn't generate a customized recipe right now.";
                
                // Simple markdown-to-HTML conversion for paragraphs and H3
                let htmlText = text
                    .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
                    .replace(/###\s*(.*)/g, '<h3>$1</h3>')
                    .split('\n')
                    .filter(line => line.trim() !== '')
                    .map(line => {
                        if (line.startsWith('<h3>')) return line;
                        return `<p>${line}</p>`;
                    }).join('');

                showModal(`Customization for ${recipe.name}`, htmlText);
            } catch (error) {
                console.error("Gemini Customization Error:", error);
                showModal(`Customization Failed`, `<p class="text-red-500">The recipe customization failed due to an API error. Please try again later.</p>`);
            }
        }

        async function runAnalysis(recipeId) {
            const recipe = allRecipes.find(r => r.id === recipeId);
            if (!recipe) return;

            showLoading(`Analyzing "${recipe.name}"`);

            const userQuery = `You are a pediatric nutritionist. Analyze the following baby food recipe for a ${recipe.age} month old: Recipe Name: ${recipe.name}, Ingredients: ${recipe.ingredients}, Notes: ${recipe.notes}. Summarize its 3 main nutritional benefits and identify any potential common allergens present. Be concise and use bullet points for the benefits.`;
            
            const systemPrompt = "You are a highly reliable pediatric nutritionist using Google Search to ground your findings. Provide a summary of the 3 main nutritional benefits and clearly list any common allergens. Prioritize clarity and safety. Only include citations from the Google Search grounding.";

            try {
                const payload = {
                    contents: [{ parts: [{ text: userQuery }] }],
                    tools: [{ "google_search": {} }], // Enable grounding
                    systemInstruction: { parts: [{ text: systemPrompt }] },
                };

                const options = {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                };

                const response = await retryFetch(GEMINI_API_URL, options);
                const result = await response.json();
                const candidate = result.candidates?.[0];
                let text = "Sorry, I couldn't generate the nutritional analysis right now.";

                if (candidate && candidate.content?.parts?.[0]?.text) {
                    text = candidate.content.parts[0].text;
                }
                
                // Simple markdown-to-HTML conversion
                let htmlText = text
                    .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
                    .split('\n')
                    .filter(line => line.trim() !== '')
                    .map(line => {
                        if (line.startsWith('* ') || line.startsWith('- ')) {
                            return `<li class="ml-2">${line.substring(2).trim()}</li>`;
                        }
                        return `<p>${line}</p>`;
                    }).join('');
                
                // Extract Citations
                let sources = [];
                const groundingMetadata = candidate?.groundingMetadata;
                if (groundingMetadata && groundingMetadata.groundingAttributions) {
                    sources = groundingMetadata.groundingAttributions
                        .map(attribution => ({
                            uri: attribution.web?.uri,
                            title: attribution.web?.title,
                        }))
                        .filter(source => source.uri && source.title);
                }

                showModal(`Nutritional Analysis: ${recipe.name}`, `<ul class="list-disc pl-5">${htmlText}</ul>`);

                if (sources.length > 0) {
                    let sourcesHtml = '<h4 class="text-sm font-bold mb-1">Sourced Information:</h4>';
                    sourcesHtml += '<ul class="list-disc list-inside mt-1 space-y-0.5 text-xs">';
                    sources.forEach(s => {
                        sourcesHtml += `<li><a href="${s.uri}" target="_blank" class="hover:underline text-purple-600">${s.title}</a></li>`;
                    });
                    sourcesHtml += '</ul>';
                    modalCitations.innerHTML = sourcesHtml;
                    modalCitations.classList.remove('hidden');
                }

            } catch (error) {
                console.error("Gemini Analysis Error:", error);
                showModal(`Analysis Failed`, `<p class="text-red-500">The nutritional analysis failed due to an API error. Please try again later.</p>`);
            }
        }
        // --- End Modal and Gemini Functions ---


        window.toggleDetails = toggleDetails;
        window.handleAgeSelection = handleAgeSelection;
        window.handleMealSelection = handleMealSelection;
        window.startOver = startOver;
        window.runCustomizer = runCustomizer;
        window.runAnalysis = runAnalysis;
        window.closeModal = closeModal;

        // Initialize on load
        updateUI(); 
    </script>
</body>
</html>
