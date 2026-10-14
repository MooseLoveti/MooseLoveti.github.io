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

            {% assign notes = site.pages | where_exp: "n", "n.path contains 'notes/'" | where_exp: "n", "n.path != 'notes/index.md'" %}
            {% if notes.size > 0 %}
            {% assign categories = notes | group_by: "category" | sort: "name" %}
            <nav class="note-category-nav" aria-label="カテゴリ">
                {% for category in categories %}
                <a class="tag" href="#{{ category.name | default: 'other' | slugify }}">{{ category.name | default: "その他" }}（{{ category.size }}）</a>
                {% endfor %}
            </nav>

            {% for category in categories %}
            <section class="note-category" id="{{ category.name | default: 'other' | slugify }}">
                <h3>{{ category.name | default: "その他" }}</h3>
                {% assign topics = category.items | group_by: "topic" | sort: "name" %}
                {% for topic in topics %}
                <div class="note-topic">
                    <h4>{{ topic.name | default: "その他" }}</h4>
                    {% assign topic_notes = topic.items | sort: "date" | reverse %}
                    <div class="article-list">
                        {% for note in topic_notes %}
                        <article class="article-record">
                            <span class="tag">{{ note.date | date: "%Y-%m-%d" }}{% if note.updated %}（更新 {{ note.updated | date: "%Y-%m-%d" }}）{% endif %}</span>
                            <h5><a href="{{ note.url }}">{{ note.title }}</a></h5>
                            {% if note.summary %}<p>{{ note.summary }}</p>{% endif %}
                            {% if note.tags.size > 0 %}<div class="tag-row">{% for tag in note.tags %}<span class="tag">{{ tag }}</span>{% endfor %}</div>{% endif %}
                        </article>
                        {% endfor %}
                    </div>
                </div>
                {% endfor %}
            </section>
            {% endfor %}
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
