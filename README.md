<h1 align="center">
   Computational Text Analysis of COP30 Media Coverage: Topic Modeling, SDG Classification, and Sentiment Analysis
</h1>

Data and prompts supporting the paper **"Computational Text Analysis of COP30 Media Coverage: Topic Modeling, SDG Classification, and Sentiment Analysis"**.

The study analyzes how COP30 — the 30th Conference of the Parties of the UNFCCC, held in Belém, Brazil — was represented in journalistic coverage and in social media debate. This repository holds the **news corpus** and the **LLM prompts** used in the pipeline, together with the full **variable specification** of both the news and the social media data (X/Twitter posts, Reddit submissions and comments).

> **Note:** Due to ethical considerations, the social media corpora (X/Twitter and Reddit) cannot be made publicly available. Their variable specification is documented below for transparency and reproducibility of the collection process.

---

## Repository layout

```
cop30-media-corpus/
├── README.md
├── prompts/
│   ├── filtering_prompt.txt
│   └── sentiment_prompt.txt
└── data/
    ├── cop30-news-raw.zip
    └── cop30-news-final.zip
```

---

## Variable specification

### News (`data/raw/news/*.csv`, `data/final/*.csv`)

| Column      | Type   | Description                                                    |
| ----------- | ------ | -------------------------------------------------------------- |
| `_id`       | string | MongoDB ObjectId of the record                                 |
| `Title`     | string | Article headline                                               |
| `Abstract`  | string | Summary / standfirst, when the outlet provides one             |
| `BodyText`  | string | Full article body; the input to SDG mapping and topic modeling |
| `Published` | string | Publication date as emitted by the source (see caveats)        |
| `URL`       | string | Canonical article URL — the de-duplication key                 |
| `Keyword`   | string | Query term that retrieved the article                          |
| `Platform`  | string | Publishing outlet                                              |

`data/final/dataset_final_cop30.csv` adds:

| Column                                   | Type      | Description                                                                                       |
| ---------------------------------------- | --------- | ------------------------------------------------------------------------------------------------- |
| `Title_pp`, `Abstract_pp`, `BodyText_pp` | string    | Preprocessed text: lowercased, punctuation/accents/special characters stripped, stopwords removed |
| `parecer_final`                          | string    | Aggregated relevance verdict from the LLM filter (`Sim`/`Yes` — Portuguese and English runs)      |
| `label_final`                            | list[int] | SDG numbers assigned to the article, e.g. `[1, 7, 8]`                                             |

### X / Twitter

Fields follow the `twscrape` object model. Grouped by the categories described in the paper:

| Category              | Fields                                                                                                                                                                                                                                                                                                                                                                                                   |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Post identity         | `_id`, `id`, `id_str`, `url`, `_type`                                                                                                                                                                                                                                                                                                                                                                    |
| Textual content       | `rawContent`, `lang`, `hashtags[]`, `cashtags[]`, `links[].url/.text/.tcourl`, `mentionedUsers[].id/.username/.displayname`                                                                                                                                                                                                                                                                              |
| Temporal metadata     | `date` (UTC, ISO 8601)                                                                                                                                                                                                                                                                                                                                                                                   |
| Engagement metrics    | `replyCount`, `retweetCount`, `likeCount`, `quoteCount`, `bookmarkedCount`, `viewCount`                                                                                                                                                                                                                                                                                                                  |
| Author information    | `user.id`, `user.username`, `user.displayname`, `user.rawDescription`, `user.created`, `user.followersCount`, `user.friendsCount`, `user.statusesCount`, `user.favouritesCount`, `user.listedCount`, `user.mediaCount`, `user.location`, `user.verified`, `user.blue`, `user.blueType`, `user.protected`, `user.profileImageUrl`, `user.profileBannerUrl`, `user.descriptionLinks[]`, `user.pinnedIds[]` |
| Threading / structure | `conversationId`, `conversationIdStr`, `inReplyToTweetId`, `inReplyToTweetIdStr`, `inReplyToUser.*`, `quotedTweet.*`, `retweetedTweet.*`                                                                                                                                                                                                                                                                 |
| Media                 | `media.photos[].url`, `media.videos[].thumbnailUrl/.duration/.views/.variants[]`, `media.animated[].thumbnailUrl/.videoUrl`                                                                                                                                                                                                                                                                              |
| Link preview card     | `card.title`, `card.description`, `card.url`, `card.vanityUrl`, `card.photo.url`, `card.video.*`, `card.options[].label/.votesCount`, `card.finished` (poll cards)                                                                                                                                                                                                                                       |
| Geolocation           | `place.id`, `place.fullName`, `place.name`, `place.type`, `place.country`, `place.countryCode`, `coordinates.latitude`, `coordinates.longitude`                                                                                                                                                                                                                                                          |
| Client                | `source`, `sourceUrl`, `sourceLabel`, `possibly_sensitive`                                                                                                                                                                                                                                                                                                                                               |

`quotedTweet.*` and `retweetedTweet.*` recursively repeat the whole schema for the referenced post (nested up to `quotedTweet.quotedTweet.*`). Geolocation is populated for only a small minority of posts.

### Reddit submissions

| Column                     | Type     | Description                                           |
| -------------------------- | -------- | ----------------------------------------------------- |
| `_id`                      | string   | MongoDB ObjectId                                      |
| `id`                       | string   | Reddit submission id (base36) — join key for comments |
| `titulo`                   | string   | Submission title                                      |
| `autor`                    | string   | Author username; `[deleted]` when removed             |
| `subreddit`                | string   | Subreddit the submission was posted to                |
| `texto`                    | string   | Selftext body; empty for link posts                   |
| `url`                      | string   | Target URL (external link, or the submission itself)  |
| `score`                    | int      | Net upvotes at collection time                        |
| `upvote_ratio`             | float    | Share of votes that were upvotes, 0–1                 |
| `num_comentarios`          | int      | Comment count reported by the API                     |
| `criado_em`                | datetime | Creation timestamp                                    |
| `link_post`                | string   | Permalink                                             |
| `palavra_chave_encontrada` | string   | Seed term that matched this submission                |

### Reddit comments

| Column            | Type     | Description                                                         |
| ----------------- | -------- | ------------------------------------------------------------------- |
| `_id`             | string   | MongoDB ObjectId                                                    |
| `id`              | string   | Reddit comment id                                                   |
| `post_id`         | string   | Parent submission id → `Reddit.posts_reddit.id`                     |
| `autor`           | string   | Author username; `[deleted]` when removed                           |
| `texto`           | string   | Comment body; `[removed]` / `[deleted]` for moderated content       |
| `score`           | int      | Net upvotes at collection time                                      |
| `criado_em`       | datetime | Creation timestamp                                                  |
| `nivel`           | int      | Thread depth; `0` = top-level reply to the submission               |
| `parent_id`       | string   | Reddit fullname of the parent (`t3_` = submission, `t1_` = comment) |
| `link_comentario` | string   | Permalink                                                           |
| `editado`         | bool     | Whether the comment was edited                                      |

---

## Prompts

Both prompts were run at `temperature = 0` in a zero-shot setting.

- **`prompts/filtering_prompt.txt`** — binary relevance classification of Brazilian news articles (`Sim`/`Não`), returning JSON only. Executed against three models via OpenRouter: `gemini-flash-2.0`, `liquid-7b` and `qwen3-8b`. Inter-model agreement: Fleiss' κ = 0.6633. Disagreements were reconciled with the Dawid–Skene algorithm, then a random sample (95% CI, 10% margin of error) was manually validated.
- **`prompts/sentiment_prompt.txt`** — three-class sentiment labelling (Positive / Negative / Neutral). Run on `Llama 3:8B` locally through Ollama with `temperature = 0`, `top_p = 1.0`, `top_k = 1`, `seed = 42`. Its output was combined with language-specific RoBERTa models (pysentimiento) under a strict unanimity rule; disagreements were labelled _Divergent_ and dropped. Cohen's κ = 0.4374.

The prompt files reproduce the wording used in the runs. `filtering_prompt.txt` retains its original Portuguese output labels (`'Sim'`/`'Não'`); the paper presents the translated version.

---

## Citation

```bibtex
Soon
```

## Authors

<table>
  <tr>
    <td align="center">
      <a href="">
        <img src="https://avatars.githubusercontent.com/u/88400274?v=4" width="100px;" alt="Foto do Jonathan"/><br>
        <sub>
          <b>Jonathan O. Fernandez</b><br>
          UEMA – São Luís, Brasil
        </sub>
      </a>
    </td>
    <td align="center">
      <a href="">
        <img src="https://media.licdn.com/dms/image/v2/D4D03AQGr7VEMedK80Q/profile-displayphoto-shrink_800_800/B4DZU8.BzcHAAc-/0/1740484648769?e=1789603200&v=beta&t=iYD4ZaWGg9PNV57H9WZTJp33_7by54ULlr8vpr2HzVY" width="100px;" alt="Foto da Bianca"/><br>
        <sub>
          <b>Bianca C. Leão</b><br>
          IFMA – São Luís, Brasil
        </sub>
      </a>
    </td>
    <td align="center">
    <td align="center">
      <a href="">
        <img src="https://laca-ufopa.com.br/public/images/current/andrey.png" width="100px;" alt="Foto do Andrey"/><br>
        <sub>
          <b>Andrey S. Pontes</b><br>
          UFOPA – Santarém, Brasil
        </sub>
      </a>
    </td>
    <td align="center">
      <a href="">
        <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSgP0DVQBNiYVOmGw5LwQM0gh8F44jIFg1q33a3HWFIjOig5be7JmsUetbI&s=10" width="100px;" alt="Foto do Jacob"/><br>
        <sub>
          <b>Antonio F. L. Jacob Junior</b><br>
          UEMA – São Luís, Brasil
        </sub>
      </a>
    </td>
    <td align="center">
      <a href="">
        <img src="https://avatars.githubusercontent.com/u/42838538?s=400&v=4" width="100px;" alt="Foto do Fábio"/><br>
        <sub>
          <b>Fábio M. F. Lobato</b><br>
          USP – São Carlos, Brasil
        </sub>
      </a>
    </td>
  </tr>
</table>
