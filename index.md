<style>
.tabs {
  display: flex;
  flex-direction: column;
  width: 100%;
  font-family: sans-serif;
}
.tab-labels {
  display: flex;
  gap: 10px;
}
.tab-labels label {
  padding: 10px 20px;
  background: #eee;
  cursor: pointer;
  border-radius: 5px 5px 0 0;
}
input[type="radio"] {
  display: none;
}
.tab-content {
  display: none;
  padding: 20px;
  border: 1px solid #ccc;
  border-top: none;
  background: #f9f9f9;
}
input#tab1:checked ~ .tab-labels label[for="tab1"],
input#tab2:checked ~ .tab-labels label[for="tab2"],
input#tab3:checked ~ .tab-labels label[for="tab3"] {
  background: #fff;
  border-bottom: 1px solid #fff;
}
input#tab1:checked ~ .contents #content1,
input#tab2:checked ~ .contents #content2,
input#tab3:checked ~ .contents #content3 {
  display: block;
}
</style>

<div class="tabs">
  <input type="radio" name="tabset" id="tab1" checked>
  <input type="radio" name="tabset" id="tab2">
  <input type="radio" name="tabset" id="tab3">

  <div class="tab-labels">
    <label for="tab1">Tab 1</label>
    <label for="tab2">Tab 2</label>
    <label for="tab3">Tab 3</label>
  </div>

  <div class="contents">
    <div id="content1" class="tab-content">
      <h3>Welcome to Tab 1</h3>
      <p>This is the content of the first tab.</p>
    </div>
    <div id="content2" class="tab-content">
      <h3>Tab 2 Activated</h3>
      <p>Here’s what’s in the second tab.</p>
    </div>
    <div id="content3" class="tab-content">
      <h3>Last Tab</h3>
      <p>Third tab content right here.</p>
    </div>
  </div>
</div>
