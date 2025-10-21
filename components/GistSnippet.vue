<template>
    <div class="gist mb-5" v-if="dataFetched">
        
        <!-- head info -->
        <div class="flex justify-between my-2">

            <div class="flex">
                <!-- avatar -->
                <img :src="gist.owner.avatar_url" alt="" class="w-8 h-8 rounded-full mr-2">
    
                <!-- username & gist date info -->
                <div class="flex flex-col">
                    <a id="username" :href="'https://github.com/' + gist.owner.login" target="_blank" class="font-roboto_mono_bold text-purple-text text-xs pb-1 hover:cursor-pointer">
                        @{{ gist.owner.login }}
                    </a>
                    <p class="font-roboto_mono_regular text-xs text-menu-text">Created {{ timeAgo }}</p>
                </div>
            </div>

            <!-- details and stars -->
            <div class="flex text-menu-text font-roboto_mono_regular text-xs justify-self-end lg:mx-2">
                <div class="flex lg:mx-2 hover:cursor-pointer hover:text-white">
                    <img src="/icons/gist/comments.svg" alt="" class="w-4 h-4 mr-2">
                    <span @click="showComment(gist.id)">details</span>
                </div>
                <div class="hidden lg:flex hover:cursor-pointer hover:text-white">
                    <img src="/icons/gist/star.svg" alt="" class="w-4 h-4 mx-2">
                    <span class="">stars</span>
                </div>
            </div>
            
        </div>

    <highlightjs class="snippet-container" :code="content"/>
    <div :id="'comment' + gist.id" class="hidden text-menu-text font-roboto_mono_regular mt-4 pt-4 border-top">
            <div class="flex justify-between items-center mb-2">
                <p class="text-xs">Comments ({{ comments.length }})</p>
                <img src="/icons/close.svg" alt="" class="hover:cursor-pointer" @click="showComment(gist.id)">
            </div>
            <div v-if="comments.length" class="comments-list max-h-60 overflow-y-auto pr-2">
                <div v-for="(c, idx) in comments" :key="idx" :class="['py-2', {'border-top mt-2': idx>0}]">
                    <p class="whitespace-pre-wrap break-words">{{ c }}</p>
                </div>
            </div>
            <p v-else>No comments.</p>
        </div>
    </div>
</template>

<style>
.snippet-container {
    background-color: #011221;
    padding: 5px;
    border-radius: 15px;
    border: 1px solid #1E2D3D;
    font-size: 12px;
    overflow-y: auto;
    overflow-x: auto;
    max-height: 220px;
}

.snippet-container pre {
    margin: 0;
    width: 100%;
}

.snippet-container code {
    white-space: pre; /* keep long lines on one line to enable horizontal scroll */
    width: max-content; /* let content define width so container can scroll horizontally */

}

/* show scrollbars for clarity */

pre code.hljs{
    display:block;
    /* overflow-x:auto; */
    padding:1.5em
}

code.hljs{
    padding:3px 5px
}

/* comments panel: target dynamic ids like comment<gist.id> */
[id^="comment"] {
    font-size: 13px;
}

#username:hover {
    color: #5e6ef2;
}

/* #comment {
    
} */

.hljs{color:#85a9ce;background:#011221}.hljs-doctag,.hljs-keyword,.hljs-meta .hljs-keyword,.hljs-template-tag,.hljs-template-variable,.hljs-type,.hljs-variable.language_{color:#ff7b72}.hljs-title,.hljs-title.class_,.hljs-title.class_.inherited__,.hljs-title.function_{color:#d2a8ff}.hljs-attr,.hljs-attribute,.hljs-literal,.hljs-meta,.hljs-number,.hljs-operator,.hljs-selector-attr,.hljs-selector-class,.hljs-selector-id,.hljs-variable{color:#79c0ff}.hljs-meta .hljs-string,.hljs-regexp,.hljs-string{color:#a5d6ff}.hljs-built_in,.hljs-symbol{color:#ffa657}.hljs-code,.hljs-comment,.hljs-formula{color:#8b949e}.hljs-name,.hljs-quote,.hljs-selector-pseudo,.hljs-selector-tag{color:#7ee787}.hljs-subst{color:#c9d1d9}.hljs-section{color:#1f6feb;font-weight:700}.hljs-bullet{color:#f2cc60}.hljs-emphasis{color:#c9d1d9;font-style:italic}.hljs-strong{color:#c9d1d9;font-weight:700}.hljs-addition{color:#aff5b4;background-color:#033a16}.hljs-deletion{color:#ffdcd7;background-color:#67060c}

</style>

<script>

import hljsVuePlugin from "@highlightjs/vue-plugin";
import 'highlight.js/lib/common';

export default {
    name: 'GistSnippet',
    props: {
        id: {
            type: String,
            required: true
        }
    },
    data(){
        return {
            gist: null,
            timeAgo: null,
            content: null,
            language: null,
            dataFetched: false,
            comments: []
        }
    },
    mounted(){
        fetch(`https://api.github.com/gists/${this.id}`)
            .then(response => response.json())
            .then(data => this.setValues(data))
            
    },
    methods: {
        async setValues(gist) {
        this.gist = gist
        this.timeAgo = this.getTimeAgo(gist.created_at)
        this.content = this.setSnippet(gist)
        this.language = Object.values(gist.files)[0].language
        this.dataFetched = true
    this.comments = await this.setComments(gist.comments_url)
        },
        getTimeAgo(dateStr) {
            const now = new Date()
            const then = new Date(dateStr)
            const diffMs = now.getTime() - then.getTime()
            const seconds = Math.floor(diffMs / 1000)
            const minutes = Math.floor(seconds / 60)
            const hours = Math.floor(minutes / 60)
            const days = Math.floor(hours / 24)
            if (seconds < 60) return `${seconds} seconds ago`
            if (minutes < 60) return `${minutes} minutes ago`
            if (hours < 24) return `${hours} hours ago`
            return `${days} days ago`
        },
        setSnippet(gist) {
            let snippet = Object.values(gist.files)[0].content // Object.values(gist.files)[0].filename.content
            return snippet
        },
        async setComments(comments_url){
            try {
                const response = await fetch(comments_url)
                const data = await response.json()
                if (Array.isArray(data)) {
                    return data.map(c => c?.body).filter(Boolean)
                }
            } catch (e) {
                console.log(`error fetching comments on ${comments_url}`, e)
            }
            return []
        },
        showComment(id) {
            let comment = document.getElementById('comment' + id)
            comment.classList.toggle('hidden')
        }
    },
    components: {
        highlightjs: hljsVuePlugin.component
    }
}
</script>