<svelte:head>
  <title>Contact me</title>
</svelte:head>

<h1>Contact</h1>

<form action="mailto:jesschn@mit.edu" method="GET" class="wrapper">
    <label class="labelgrid">
        Email:
        <input name="email" type="email"/>
    </label>
    

    <label class="labelgrid">
        Subject:
        <input name="subject"/>
    </label>
    

    <label class="labelgrid">
        Body:
        <textarea name="body"></textarea>
    </label>
    
    <button>Send</button>
</form>