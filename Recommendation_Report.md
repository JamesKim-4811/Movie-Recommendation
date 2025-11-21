#### **Title: Movie Recommendation System Report**

#### **1. Introduction**
Movie recommendation systems surface items a user is likely to enjoy by leveraging historical interaction data. They improve user engagement, retention, and discovery in media platforms. This project implements three approaches: user-based collaborative filtering (recommend users' favorite movies to similar users), item-based collaborative filtering (recommend movies similar to a target movie), and a graph-based random-walk (Pixie‑inspired) method that explores the user–movie bipartite graph to find related titles.

#### **2. Dataset Description**
The MovieLens 100K dataset (ml-100k) is used. It contains 943 users, 1,682 movies, and 100,000 ratings. Key fields used are user_id, movie_id, rating, and movie title. Preprocessing steps: read raw files into pandas DataFrames, rename columns consistently, merge ratings with movie titles, aggregate duplicate user–movie pairs by mean rating, and normalize ratings per user (center by user mean) for graph algorithms. A user–movie pivot matrix was created for similarity-based methods and an adjacency list graph for random-walk methods.

#### **3. Methodology**
- User-based collaborative filtering: compute cosine similarity between users on the zero-filled user–movie matrix; generate recommendations by computing a similarity‑weighted average rating for each candidate movie from other users, then exclude movies the target user has already rated.
- Item-based collaborative filtering: compute cosine similarity between items by transposing the user–movie matrix; for a given movie, select the top-N most similar movies by similarity score and map to titles.
- Random-walk (Pixie‑inspired): build a bipartite graph (users ↔ movies). Perform many short random walks starting from a seed (user node or movie node); count visits to movie nodes (excluding the seed) and rank by visit frequency. Enhancements include multiple walks, restart probability, and degree normalization to mitigate popularity bias.

#### **4. Implementation Details**
- Data loading: pandas.read_csv with appropriate separators and encodings; created `ratings`, `movies`, and `users` DataFrames and exported CSV copies.
- Similarity computation: replaced NaNs with zeros (zero_user_movie_matrix) and used sklearn.metrics.pairwise.cosine_similarity to produce `user_sim_df` and `item_sim_df`.
- recommend_movies_for_user(user_id, num): retrieves the user's similarity vector, computes similarity-weighted scores across movies, filters out already rated movies, and returns top titles in a DataFrame indexed by ranking.
- recommend_movies(movie_name, num): looks up movie_id, extracts the item similarity row, excludes the seed, and returns top similar movie titles.
- Graph construction: merged ratings with titles, grouped by (user_id, movie_id, title) averaging ratings, normalized per user, and built an adjacency dict where keys are node ids (users and movies) and values are neighbor sets.
- Random walks: implemented simple_random_walk and weighted_pixie_recommend. Each walk samples a random neighbor per step; movie visit counts are aggregated and top movies returned. Seed option enables deterministic runs.

#### **5. Results and Evaluation**
- Example outputs (representative):
  ![User Based](Movie-Recommendation/images/user-cf.png "User-based")
  - User-based: recommend_movies_for_user(10, num=5) → top 5 movie titles recommended by similar users.
  ![Item Based](Movie-Recommendation/images/item-cf.png "Item-based")
  - Item-based: recommend_movies("Jurassic Park (1993)", num=5) → 5 most similar movies by cosine similarity.
  ![Pixie random-walk](Movie-Recommendation/images/pixie.png "Pixie random-walk")
  - Pixie random-walk: weighted_pixie_recommend("Jurassic Park (1993)", walk_length=100, num=5, seed=42) → top movies ranked by visit counts.
- Comparison:
  - User-based CF personalizes well when users have rich rating histories, but suffers from sparsity and cold-start users.
  - Item-based CF is more stable for popular items and scales well for retrieval of similar titles, but can reflect popularity bias.
  - Random-walk (Pixie) captures transitive, graph-structural relationships and can surface serendipitous recommendations; however, it is stochastic and biased toward high-degree nodes unless normalized.
- Limitations:
  - Exact title matching required for item-based function.
  - Naive zero-filling affects similarity quality; mean-centering or normalization can improve results.
  - Single short walks are noisy; results improve with many walks or aggregation.
  - No held-out evaluation (RMSE / precision@K) implemented in this notebook — evaluation would quantify accuracy.

#### **6. Conclusion**
The project implements and compares three complementary recommendation approaches. User-based CF offers personalization from peer preferences, item-based CF provides robust similar-item retrieval, and Pixie‑inspired random walks uncover structural relationships in the interaction graph. Improvements include hybridization (combine item and graph signals), rating normalization, minimum co-rating thresholds, degree-normalized walk scoring, and an offline evaluation pipeline (train/test split with precision/recall or NDCG). Real-world deployments can use item similarity for "more like this" widgets, user CF for personalized feeds, and Pixie-style graph walks for fast, interpretable discovery and multi-seed personalization.