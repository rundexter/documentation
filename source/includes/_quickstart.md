# Quickstart

Before getting started, you’ll need Node and git installed.<br/>

    **For OSX:** <a href="https://nodejs.org/dist/v4.2.1/node-v4.2.1.pkg">Node JS</a> | <a href="https://git-scm.com/download/mac">git</a> 
    
    **For Linux:** <a href="https://nodejs.org/en/download/">Node JS</a> | <a href="https://git-scm.com/download/linux">git</a> 
    
    **For Windows:** You’ll need to use a Linux virtual machine and follow the links above. <a href="https://www.virtualbox.org/manual/ch01.html#intro-installing" target="_blank">Here’s a guide</a> on how to go about that.

    **Step 1**    
<block class="highlight shell">
 <code>
  <span class="gp">$ </span>npm install -g rundexter
 </code>
</block>

    **Step 2a** <br/>
<block class="highlight shell">
 <code>
  #If you don’t have a key, or are unsure, run this     
  #warning: if prompted to overwrite, choose “no.”<br/>
  <span class="gp">$ </span>ssh-keygen -f ~/.ssh/id_rsa -N ""
 </code>
</block>
    
    **Step 2b** <br/>
<block class="highlight shell">
 <code>
  <span class="gp">$ </span>dexter login [username or email]
 </code>
</block>

    **Step 3**<br/>
<block class="highlight shell">
 <code>
  <span class="gp">$ </span>dexter create “my first module”
 </code>
</block>

    **Step 4**<br/>
<block class="highlight shell">
 <code>
  <span class="gp">$ </span>git commit -am “my first edit”
 </code>
</block>

    **Step 5**<br/>
<block class="highlight shell">
 <code>
  <span class="gp">$ </span>dexter push
 </code>
</block>

    **Step 6**<br/>
    Congrats! You’ve created your first module. It’s in `./my-first-module` Check out `meta.json` and `index.js` to get started. For anything else, let's get started, shall we?