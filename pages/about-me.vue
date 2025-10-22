<template>
  <main v-if="!loading" id="about-me" class="page">

    <div id="mobile-page-title">
      <h2>_about-me</h2>
    </div>

  <div id="page-menu" class="w-full flex lg:flex-none" :style="pageMenuStyle">

      <!-- DESKTOP section icons -->
      <div id="sections">
        <div id="section-icon" v-for="section in config.about.sections" :key="section.title" :class="{ active: isSectionActive(section.title)}">
          <img :id="'section-icon-' + section.title" :src="section.icon" :alt="section.title + '-section'" @click="focusCurrentSection(section)">
        </div>
        <!-- Code Snippet toggle icon (desktop only) -->
        <div id="section-icon" :class="{ active: showGists }" class="hidden lg:flex justify-center my-6">
          <img id="section-icon-code-snippet" src="/icons/gist/code-snippet.svg" alt="code-snippet-section" class="w-6 h-6" @click="toggleGists" />
        </div>
      </div>

      <!-- focused section content -->
  <div id="section-content" class="hidden lg:block w-full h-full border-right">

        <!-- title -->
        <div id="section-content-title" class="hidden lg:flex items-center min-w-full">
          <img id="section-arrow-menu" src="/icons/arrow.svg" alt="" class="section-arrow mx-3 open">
          <p v-html="config.about.sections[currentSection]?.title" class="font-roboto_mono_regular text-white text-sm"></p>
        </div>

        <!-- folders -->
        <div>
          <div v-for="(folder, key, index) in config.about.sections[currentSection]?.info" :key="key" class="grid grid-cols-2 items-center my-2 font-roboto_mono_regular text-menu-text">
            <!-- folder header: click selects folder; arrow toggles expand -->
            <div class="flex col-span-2 hover:text-white hover:cursor-pointer" @click="focusCurrentFolder(folder)">
              <img id="diple" src="/icons/diple.svg" alt="" :class="{ open: openFolders[folder.title]}" @click.stop="toggleFolder(folder.title)">
              <img :src="'/icons/folder' + (index+1) + '.svg'" alt="" class="mr-3">
              <p :id="folder.title" v-html="key" :class="{ active: isActive(folder.title)}"></p>
            </div>
            <!-- files list (only if folder has files and is open) -->
            <div v-if="folder.files !== undefined && openFolders[folder.title]" class="col-span-2">
              <div v-for="(file, fkey) in folder.files" :key="fkey" class="hover:text-white hover:cursor-pointer flex my-2" @click="selectFile(folder.title, fkey)">
                <img src="/icons/markdown.svg" alt="" class="ml-8 mr-3"/>
                <p :class="{ 'text-white': isFileActive(folder.title, fkey) }">{{ fkey }}</p>
              </div> 
            </div>
          </div>
        </div>

        <!-- contact -->
        <div id="section-content-title-contact" class="flex items-center min-w-full border-top">
          <img id="section-arrow-menu" src="/icons/arrow.svg" alt="" class="section-arrow mx-3 open">
          <p v-html="config.contacts.direct.title" class="font-roboto_mono_regular text-white text-sm"></p>
        </div>
        <div id="contact-sources" class="hidden lg:flex lg:flex-col my-2 pr-2">
          <div v-for="(source, key) in config.contacts.direct.sources" :key="key" class="flex items-center mb-2">
            <img :src="'/icons/' + key + '.svg'" alt="" class="mx-4">
            <a v-html="source" href="/" class="font-roboto_mono_regular text-menu-text hover:text-white break-all"></a>
          </div>
        </div>

      </div>

      <!-- mobile -->
  <div id="section-content" class="lg:hidden w-full font-roboto_mono_regular">

        <div v-for="section in config.about.sections" :key="section.title">
          
          <!-- section title (mobile) -->
          <div :key="section.title" :src="section.icon" id="section-content-title" class="flex lg:hidden mb-1" @click="focusCurrentSection(section)">
            <img src="/icons/arrow.svg" :id="'section-arrow-' + section.title" alt="" class="section-arrow">
            <p v-html="section.title" class=" text-white text-sm"></p>
          </div>

          <!-- folders -->
          <div :id="'folders-' + section.title" class="hidden"> <!-- <div :id="'folders-' + section.title" :class="currentSection == section.title ? 'block' : 'hidden'"> -->
            <div v-for="(folder, key, index) in config.about.sections[section.title]?.info" :key="key" class="grid grid-cols-2 items-center my-2 font-roboto_mono_regular text-menu-text hover:text-white hover:cursor-pointer">
              <!-- folder header: click selects folder; arrow toggles expand -->
              <div class="flex col-span-2" @click="focusCurrentFolder(folder)">
                <img id="diple" src="/icons/diple.svg" :class="{ open: openFolders[folder.title]}" @click.stop="toggleFolder(folder.title)">
                <img :src="'icons/folder' + (index+1) + '.svg'" alt="" class="mr-3">
                <p :id="folder.title" v-html="key" :class="{ active: isActive(folder.title)}"></p>
              </div>
              <!-- files list (only if folder has files and is open) -->
              <div v-if="folder.files !== undefined && openFolders[folder.title]" class="col-span-2">
                <div v-for="(file, fkey) in folder.files" :key="fkey" class="hover:text-white hover:cursor-pointer flex my-2" @click="selectFile(folder.title, fkey)">
                  <img src="/icons/markdown.svg" alt="" class="ml-8 mr-3"/>
                  <p :class="{ 'text-white': isFileActive(folder.title, fkey) }">{{ fkey }}</p>
                </div>
              </div>
            </div>
          </div>
          
        </div>

        <!-- section content title -->
        <div id="section-content-title" class="flex items-center min-w-full" @click="showContacts()">
          <img src="/icons/arrow.svg" alt="" id="section-arrow" class="section-arrow">
          <p v-html="config.contacts.direct.title" class="font-roboto_mono_regular text-white text-sm"></p>
        </div>

        <!-- section content folders -->
        <div id="contacts" class="hidden">
          <div v-for="(source, key) in config.contacts.direct.sources" :key="key" class="flex items-center my-2">
            <img :src="'/icons/' + key + '.svg'" alt="">
            <a v-html="source" href="/" class="font-roboto_mono_regular text-menu-text hover:text-white ml-4"></a>
          </div>
        </div>

      </div>

    </div>
    <!-- Drag handle between left menu and content (desktop only) -->
    <div id="resize-handle"
      class="hidden lg:block w-1 cursor-col-resize bg-transparent hover:bg-[#1E2D3D] border-right select-none"
      @mousedown="startResize"
    ></div>
    <!-- MENU END -->

    <!-- content: resizable split between left (about) and right (gists) -->
  <div id="content-split" class="flex flex-col lg:flex-row h-full w-full" :style="contentSplitStyle" :class="{ 'gists-hidden': !showGists }">
      
      <!-- LEFT PANE -->
      <div id="left" class="w-full lg:flex-none flex flex-col border-right">
        
        <!-- windows tab desktop -->
        <div class="tab-height w-full hidden lg:flex border-bot items-center">
          <div class="flex items-center border-right h-full">
            <p v-html="config.about.sections[currentSection]?.title" class="font-roboto_mono_regular text-menu-text text-sm px-3"></p>
            <img src="/icons/close.svg" alt="" class="mx-3">
          </div>
        </div>

        <!-- windows tab mobile -->
        <div id="tab-mobile" class="flex lg:hidden font-roboto_mono_regular">
          <span class="text-white">// </span>
          <h3 v-html="config.about.sections[currentSection]?.title" class="text-white px-2"></h3>
          <span class="text-menu-text"> / </span>
          <h3 v-html="config.about.sections[currentSection]?.info[folder].title" class="text-menu-text pl-2"></h3>
        </div>
        
        <!-- text -->
        <div id="commented-text" class="flex h-full w-full lg:border-right overflow-hidden">

          <div class="w-full h-full ml-5 mr-10 py-4 lg:py-6 overflow-scroll">
            <ClientOnly>
              <!-- Prefer Nuxt Content-rendered Markdown when available -->
              <ContentRenderer v-if="bio" :value="bio" />
              <!-- Fallback to legacy string-based renderer -->
              <CommentedText v-else :text="displayText" />
            </ClientOnly>
          </div>
          
          <!-- scroll bar -->
          <div id="scroll-bar" class="h-full border-left hidden lg:flex justify-center py-1 flex-none">
            <div id="scroll"></div>
          </div>

        </div>

      </div>

      <!-- SPLITTER HANDLE -->
      <div id="content-resize-handle"
        class="hidden lg:block w-1 cursor-col-resize bg-transparent hover:bg-[#1E2D3D] border-right select-none"
        @mousedown="startContentResize"
        v-show="showGists"
      ></div>

      <!-- RIGHT PANE -->
  <div id="right" class="max-w-full flex-1 min-w-0 flex flex-col" v-show="showGists">
        
        <!-- windows tab for gists (desktop) -->
        <div class="tab-height w-full h-full hidden lg:flex border-bot items-center" v-if="showGists">
          <div class="flex items-center border-right h-full">
            <p class="font-roboto_mono_regular text-menu-text text-sm px-3">code-snippets</p>
            <img src="/icons/close.svg" alt="" class="mx-3 hover:cursor-pointer" @click="closeGists">
          </div>
        </div>

        <!-- windows tab mobile -->
        <div class="tab-height w-full h-full flex-none lg:hidden items-center"></div>

        <div id="gists-content" class="flex min-w-0">
          <div id="gists" class="flex flex-col lg:px-6 lg:py-4 flex-1 min-w-0 lg:min-w-[340px] lg:max-w-[1280px] overflow-visible">
            <!-- title -->
            <h3 class="text-white lg:text-menu-text mb-4 text-sm"># Code snippet showcase:</h3>

            <div class="flex flex-col overflow-scroll min-w-0">
              <!-- snippets -->
              <GistSnippet data-aos="fade-down" v-for="(gist, key) in config.gists" :key="key" :id="gist" />
            </div>
          </div>

          <!-- scroll bar -->
          <div id="scroll-bar" class="h-full border-left hidden lg:flex justify-center py-1 flex-none">
            <div id="scroll"></div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<style>

#sections {
  width: 5rem; /* 80px */
  height: 100%;
  display: none;
  border-right: 1px solid #1E2D3D;
}

/* LG */
@media (min-width: 1024px) {
  #sections {
    display: block;
  }
}

#section-icon {
  @apply my-6 hover:cursor-pointer flex justify-center;
  opacity: 0.4;
}

#section-icon.active {
  opacity: 1;
}

#section-icon:hover {
  opacity: 1;
}

.tab-height {
  min-height: 35px;
  max-height: 35px;
}

#tab-mobile {
  padding: 25px 20px 0px 25px;
  align-items: flex-end;
}

#scroll-bar{
  width: 20px;
}

#scroll {
  width: 14px;
  height: 7px;
  background-color: #607B96;
}

#diple {
  @apply mx-3 w-2 max-w-fit;
}

.open {
  transform: rotate(90deg);
}

.active {
  color:white;
}

#right, #left {
  height: 100%;
  overflow: hidden;
}

/* Allow right pane to show inner element scrollbars (snippets) */
#right {
  overflow: visible;
}

#gists-content {
  height: 100%;
  /* allow inner elements to render fully; snippet scrollbars are within their own boxes */
  overflow: visible;
}

@media (max-width: 1024px) {
  #gists-content {
    height: 100%;
    padding: 0px 25px;
    overflow: hidden;
  }

  #about {
  min-height: stretch;
}
}

.section-arrow {
  transition: 0.1s;
}

#section-content #contacts {
  padding: 0px 25px;
}

/* Resizable left menu (desktop) using CSS variable --pmw */
@media (min-width: 1024px) {
  #page-menu {
    width: var(--pmw, 275px) !important;
    min-width: var(--pmw, 275px) !important;
    max-width: var(--pmw, 275px) !important;
  }
  #resize-handle {
    min-height: 100%;
  }
}

/* Safety: wrap long contact text like emails */
#contact-sources a {
  word-break: break-word;
}

/* Resizable content split (desktop) using CSS variable --clw for left pane width */
@media (min-width: 1024px) {
  #content-split {
    min-height: 0;
  }
  #content-split #left {
    width: var(--clw, 48%);
    min-width: var(--clw, 48%);
    max-width: var(--clw, 48%);
    /* prevent flex from stretching it */
    flex: none;
  }
  #content-resize-handle {
    min-height: 100%;
  }
  /* When gists are hidden, expand left to full width */
  #content-split.gists-hidden #left {
    width: 100%;
    min-width: 100%;
    max-width: 100%;
    flex: 1;
  }
}

</style>

<!-- removed script setup to avoid clashing with Options API below -->

<script>
// Load raw Markdown files from /content to display in CommentedText
const mdLoaders = import.meta.glob('/content/about/**/*.md', { query: '?raw', import: 'default' });
const mdLoaders = import.meta.glob('/content/about/**/*.md', { query: '?raw', import: 'default' });
import DevConfig from '~/developer.json';
export default {
  data() {
    return {
      currentSection: 'personal-info',
      folder: 'bio',
      // Selected file key within the current folder (if any)
      selectedFileKey: null,
      // Track expanded/collapsed state per folder
      openFolders: {},
      loading: true,
      // Right pane visibility (gists)
      showGists: true,
      // Resizable left panel (page menu)
      pageMenuWidth: 275,
      minWidth: 220,
      maxWidth: 480,
      isResizing: false,
      startX: 0,
      startWidth: 275,
      // Resizable content split (left about vs right gists)
      contentLeftWidth: 560,
      contentMinLeft: 320,
      contentMinRight: 360,
      isContentResizing: false,
      contentStartX: 0,
      contentStartWidth: 560,
      // Loaded HTML from markdown (if available)
      currentHtml: '',
    }
  },
  /**
   * In setup we can define the data we want to use in the component before the component is created.
   */
  setup() {
    // Fetch markdown doc for bio using Nuxt Content
    const { data: bio } = useAsyncData('about-bio', () =>
      queryCollection('content').path('/about/personal-info/bio').first()
    )

    // Apply SEO tags from markdown front-matter if available
    useSeoMeta({
      title: () => bio.value?.title,
      description: () => bio.value?.description
    })

    return {
      config: DevConfig,
      bio
    }
  },
  computed: {
    // Inline CSS variable to drive width with !important override in style block
    pageMenuStyle() {
      // Use CSS custom property to override global !important min/max in tailwind.css
      return {
        '--pmw': this.pageMenuWidth + 'px'
      }
    },
    // Inline CSS variable for content split left width
    contentSplitStyle() {
      return {
        '--clw': this.contentLeftWidth + 'px'
      }
    },
    // Set active class to current page link
    isActive() {
      return folder => this.folder === folder;
    },
    isSectionActive() {
      return section => this.currentSection === section;
    },
    isOpen() {
      return folder => this.folder === folder;
    },
    // Current folder object
    currentFolderObj() {
      return this.config?.about?.sections?.[this.currentSection]?.info?.[this.folder] || {};
    },
    // Text to show: selected file content or folder description
    currentText() {
      const f = this.currentFolderObj;
      if (this.selectedFileKey && f.files && f.files[this.selectedFileKey]) {
        return f.files[this.selectedFileKey];
      }
      return f.description || '';
    },
    // Prefer Markdown HTML if loaded; otherwise fallback to JSON description
    displayText() {
      return this.currentHtml || this.currentText;
    },
    // Is a specific file active
    isFileActive() {
      return (folderTitle, fileKey) => this.folder === folderTitle && this.selectedFileKey === fileKey;
    },
  },
  methods: {
    startResize(e) {
      // Only enable on desktop
      if (window.innerWidth < 1024) return;
      this.isResizing = true;
      this.startX = e.clientX;
      this.startWidth = this.pageMenuWidth;
      document.body.style.userSelect = 'none';
      window.addEventListener('mousemove', this.onResize);
      window.addEventListener('mouseup', this.stopResize);
    },
    onResize(e) {
      if (!this.isResizing) return;
      const delta = e.clientX - this.startX;
      const next = Math.min(this.maxWidth, Math.max(this.minWidth, this.startWidth + delta));
      this.pageMenuWidth = next;
    },
    stopResize() {
      if (!this.isResizing) return;
      this.isResizing = false;
      document.body.style.userSelect = '';
      window.removeEventListener('mousemove', this.onResize);
      window.removeEventListener('mouseup', this.stopResize);
    },
    // Content splitter handlers
    startContentResize(e) {
      if (window.innerWidth < 1024) return;
      this.isContentResizing = true;
      this.contentStartX = e.clientX;
      this.contentStartWidth = this.contentLeftWidth;
      document.body.style.userSelect = 'none';
      window.addEventListener('mousemove', this.onContentResize);
      window.addEventListener('mouseup', this.stopContentResize);
    },
    onContentResize(e) {
      if (!this.isContentResizing) return;
      const container = document.getElementById('content-split');
      const containerWidth = container ? container.clientWidth : 0;
      const delta = e.clientX - this.contentStartX;
      let proposed = this.contentStartWidth + delta;
      const maxLeft = Math.max(this.contentMinLeft, containerWidth - this.contentMinRight);
      proposed = Math.min(maxLeft, Math.max(this.contentMinLeft, proposed));
      this.contentLeftWidth = proposed;
    },
    stopContentResize() {
      if (!this.isContentResizing) return;
      this.isContentResizing = false;
      document.body.style.userSelect = '';
      window.removeEventListener('mousemove', this.onContentResize);
      window.removeEventListener('mouseup', this.stopContentResize);
      // persist
      try { localStorage.setItem('about_content_left_width', String(this.contentLeftWidth)); } catch {}
    },
    // Toggle gists right pane
    toggleGists() {
      this.showGists = !this.showGists;
    },
    closeGists() {
      this.showGists = false;
    },
    focusCurrentSection(section) {
      this.currentSection = section.title
      this.folder = Object.keys(section.info)[0]

      document.getElementById('folders-' + section.title).classList.toggle('hidden') // show folders
      document.getElementById('section-arrow-' + section.title).classList.toggle('rotate-90'); // rotate arrow
      // reset selection tracking
      this.selectedFileKey = null
      this.openFolders = {}
      this.loadCurrentContent();
    },
    focusCurrentFolder(folder) {
      this.folder = folder.title
      // handle if folder belongs to the current section. It happens when you click on a folder from a different section in mobile view.
      this.currentSection = this.config.about.sections[this.currentSection].info[folder.title] ? this.currentSection : Object.keys(this.config.about.sections).find(section => this.config.about.sections[section].info[folder.title])
      // when focusing a folder directly, clear selected file
      this.selectedFileKey = null
      this.loadCurrentContent();
    },
    /**
     * TODO: Hay que crear un método para que cuando se haga click en un folder, se muestren los archivos que contiene. Y si se hace click en un archivo, se muestre el contenido del archivo.
     * TODO:  Además de girar el icono del diple.
     */
    toggleFiles() {
      document.getElementById('file-' + this.folder).classList.toggle('hidden')
    },
    // Desktop: toggle folder expand/collapse in focused section content
    toggleFolder(folderTitle) {
      this.openFolders = { ...this.openFolders, [folderTitle]: !this.openFolders[folderTitle] };
    },
    // When user clicks a file, display its content
    selectFile(folderTitle, fileKey) {
      // focus folder if not already
      this.folder = folderTitle
      this.selectedFileKey = fileKey
      this.loadCurrentContent();
    },
    // Resolve candidate markdown paths for current selection
    resolveContentCandidates() {
      const section = this.currentSection;
      const folder = this.folder;
      const file = this.selectedFileKey;
      const candidates = [];
      if (file) {
        candidates.push(`/content/about/${section}/${folder}/${file}.md`);
      }
      // folder-level files
      candidates.push(`/content/about/${section}/${folder}.md`);
      candidates.push(`/content/about/${section}/${folder}/index.md`);
      return candidates;
    },
    // Minimal Markdown to HTML converter for headings and newlines (keeps footprint small)
    convertMdToHtml(md) {
      if (!md) return '';
      let html = md
        .replace(/^###\s+(.+)$/gm, '<h3>$1</h3>')
        .replace(/^##\s+(.+)$/gm, '<h2>$1</h2>')
        .replace(/^#\s+(.+)$/gm, '<h1>$1</h1>');
      // Convert double newlines to paragraph breaks
      html = html.replace(/\n\n+/g, '<br><br>');
      // Convert single newlines to <br>
      html = html.replace(/(?<!<br>)\n/g, '<br>');
      return html;
    },
    async loadCurrentContent() {
      try {
        const candidates = this.resolveContentCandidates();
        let raw = '';
        for (const p of candidates) {
          if (mdLoaders[p]) {
            raw = await mdLoaders[p]();
            break;
          }
        }
        this.currentHtml = raw ? this.convertMdToHtml(raw) : '';
      } catch (e) {
        this.currentHtml = '';
      }
    },
    /* Mobile */
    showContacts() {
      document.getElementById('contacts').classList.toggle('hidden')
      document.getElementById('section-arrow').classList.toggle('rotate-90'); // rotate arrow
    },
  },
  mounted(){
    this.loading = false
    // Ensure initial width aligns with CSS default (275)
    this.pageMenuWidth = Math.min(this.maxWidth, Math.max(this.minWidth, this.pageMenuWidth));
    // Load persisted content width if any
    try {
      const saved = Number(localStorage.getItem('about_content_left_width'))
      if (!Number.isNaN(saved) && saved > 0) {
        this.contentLeftWidth = saved
      }
    } catch {}
    // Default: show gists right pane on desktop, hide on mobile
    this.showGists = window.innerWidth >= 1024;
    // Initial load of content (from markdown if available)
    this.loadCurrentContent();
  },
  beforeUnmount() {
    // Clean up in case component unmounts during resize
    if (this.isResizing) {
      this.stopResize();
    }
    if (this.isContentResizing) {
      this.stopContentResize();
    }
  }
}
</script>