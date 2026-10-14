---
title: ノート一覧
layout: null
---
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ノート一覧 | Takihana Shota</title>
    <link rel="stylesheet" href="/style.css">
</head>
<body>
    <header class="site-header">
        <nav class="nav" aria-label="主要ナビゲーション">
            <a class="brand" href="/">Takihana Shota</a>
            <div class="nav-links">
                <a href="/#notes">トップに戻る</a>
            </div>
        </nav>
    </header>

    <main>
        <section class="section">
            <div class="section-head">
                <p class="label">Notes</p>
                <h2>学習ノート一覧</h2>
            </div>

            {% assign notes = site.pages | where_exp: "n", "n.path contains 'notes/'" | where_exp: "n", "n.path != 'notes/index.md'" | sort: "date" | reverse %}
            {% if notes.size > 0 %}
            <div class="article-list">
                {% for note in notes %}
                <article class="article-record">
                    <span class="tag">{{ note.date | date: "%Y-%m-%d" }}</span>
                    <h3><a href="{{ note.url }}">{{ note.title }}</a></h3>
                    {% if note.summary %}<p>{{ note.summary }}</p>{% endif %}
                    {% if note.tags.size > 0 %}<div class="tag-row">{% for tag in note.tags %}<span class="tag">{{ tag }}</span>{% endfor %}</div>{% endif %}
                </article>
                {% endfor %}
            </div>
            {% else %}
            <p>準備中</p>
            {% endif %}
        </section>
    </main>

    <footer class="site-footer">
        <small>&copy; Takihana Shota </small>
    </footer>
</body>
</html>
