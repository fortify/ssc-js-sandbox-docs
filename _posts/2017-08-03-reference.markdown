---
title:  "Reference"
date:   2017-08-03 10:50:15 -0700
---
<div class="flex-container">
{% for post in site.posts %}
{% if post.type == 'spec' %}

   <div class="flip-container vertical">
       <div class="flipper">
            <div class="front m-sm">
                <div class="flip-card-content">
                    <p class="front-title">{{post.title}}</p>
                    <div class="m-t-md">
                        <p class="card-text documents"><span>config.js:</span><span class="text-muted"> {{post.commandSub}}</span></p>
                    </div>
                    <a href="#{{post.id}}">{{post.title}}</a>
                </div>
            </div>
        </div>
    </div>
{% endif %}
{% endfor %}



