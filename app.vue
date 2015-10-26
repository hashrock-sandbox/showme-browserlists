<style>
  body {
    background: #333;
    color: white;
  }
  
  h1 {
    font-size: 2rem;
  }
  
  h2 {
    font-size: 1rem;
    text-align: center;
  }
  ul {
    list-style-type:none;
    padding: 0;
  }
  
  li {
    margin: 0 2px 2px 0;
    padding: 2px;
    font-size: 0.7rem;
  }
  li:hover{
    text-decoration: underline;
    cursor: pointer;
  }
  
  .browsers {
    display: flex;
  }
  
  .browsers__item {
    flex: 1;
  }
  
  main {
    margin: 20px;
  }
  
  .matched {
    background: #545448;
    color: #FFF2A9;
  }
  
  .InputAddOn {
    display: flex;
    margin-bottom: 1.5em;
  }
  
  .InputAddOn-field {
    flex: 1;
  }
  
  .InputAddOn-field:not(:first-child) {
    border-left: 0;
  }
  
  .InputAddOn-field:not(:last-child) {
    border-right: 0;
  }
  
  .InputAddOn-item {
    background-color: rgba(147, 128, 108, 0.1);
    color: #666666;
    font: inherit;
    font-weight: normal;
  }
  
  .InputAddOn-field,
  .InputAddOn-item {
    border: 1px solid rgba(147, 128, 108, 1);
    padding: 0.5em 0.75em;
    color: white;
  }
  
  .InputAddOn-field:first-child,
  .InputAddOn-item:first-child {
    border-radius: 2px 0 0 2px;
  }
  
  .InputAddOn-field:last-child,
  .InputAddOn-item:last-child {
    border-radius: 0 2px 2px 0;
  }
  
  .InputAddOn-field {
    font-size: 1rem;
    color: #8EFFC3;
    background: black;
  }
  .versionStrError{
    color: #FFB7B7;
  }
  
  textarea{
    background: black;
    color: white;
    width: 100%;
    height: 3rem;
    font-size: 0.8rem;    
  }
  
  button{
    background: #333;
    border: 1px solid #777;
    border-bottom: 0px;
    border-radius: 4px 4px 0 0;
    color: #9E9E9E;
    padding: 10px;
    min-width: 100px;
    outline: none;
  }
  button.selected{
    border: 1px solid #999;
    border-bottom: 0px;
    color: white;
  }
</style>

<template>
  <main>
    <h1>Show Me BrowserLists</h1>
    <div class="InputAddOn">
      <span class="InputAddOn-item">Input</span>
      <input class="InputAddOn-field" v-on="keyup: updateList(versionStr)" v-model="versionStr" v-class="versionStrError: versionStrError">
      <button class="InputAddOn-item" v-on="click: clearVersionStr">Clear</button>
    </div>
    
    <button v-on="click: view = 'cli'" v-class="selected : view === 'cli'">CLI</button>
    <button v-on="click: view = 'json'" v-class="selected : view === 'json'">JSON</button>
    <textarea v-model="cli" v-if="view === 'cli'"></textarea>
    <textarea v-model="json" v-if="view === 'json'"></textarea>
    
    <div class="browsers">
      <section v-repeat="browser: browsers" class="browsers__item">
        <h2 v-text="browser.name"></h2>
        <ul>
          <li v-repeat="version : browser.versions" v-text="version" v-class="matched: isSelected(version)" v-on="click: addVersionStr(version)"></li>
        </ul>
      </section>
    </div>
  </main>
</template>

<script>
  var browserslist = require("browserslist");

function getAllBrowsers(){
  return [
      {
        name: "IE",
        versions: browserslist("ie > 0")
      },
      {
        name: "Chrome",
        versions: browserslist("Chrome > 0")
      },
      {
        name: "Safari",
        versions: browserslist("Safari > 0")
      },
      {
        name: "FireFox",
        versions: browserslist("ff > 0")
      },
      {
        name: "iOS Safari",
        versions: browserslist("iOS > 0")
      }
    ]
}

module.exports = {
  replace: false,
  data: {
    versionStr: "last 1 version, > 10%, ie 7-8",
    browsers: [],
    selected: [],
    versionStrError: false,
    cli: "",
    json : "",
    view: "cli"
  },
  methods: {
    isSelected : function(version){
      var selected = this.selected.filter(function(item){return version === item});
      return selected.length > 0 ? true : false;
    },
    addVersionStr: function(str){
      var items = [];
      if(this.versionStr.trim() !== ""){
        items = this.versionStr.split(",").map(function(item){return item.trim()});
      }
      items.push(str);
      this.versionStr = items.join(", ");
      this.updateList(this.versionStr);
    },
    clearVersionStr: function(){
      this.versionStr = "";
      this.updateList(this.versionStr);
    },
    updateList : function(versionStr){
      var browsers;
      try {
        browsers = browserslist(versionStr.trim());
      }catch(e){
        this.versionStrError = true;
        return;
      }
      this.versionStrError = false;
      this.selected = browsers;
      
      this.cli = "npm install --global postcss-cli autoprefixer\npostcss --use autoprefixer --autoprefixer.browsers \""+ versionStr +"\" *.css -d build/";
      this.json = '{\n    "autoprefixer": {\n        "browsers": "'+versionStr+'"\n    }\n}';
    }
  },
  ready: function(){
    this.browsers = getAllBrowsers();
    this.updateList(this.versionStr);
  }
}

</script>