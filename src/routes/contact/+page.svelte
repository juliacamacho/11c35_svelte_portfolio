<svelte:head>
	<title>Contact me</title>
</svelte:head>
<h1>Contact Me</h1>
<form action="mailto:jcamac@mit.edu" method="POST" enctype="text/plain">
    <label for="email"> Email:
        <input name="email" type="email" >
    </label>

    <label for="subject"> Subject:
        <input name="subject" >
    </label>

    <label for="body"> Body:
        <textarea name="body"></textarea>
    </label>

    <button>Submit</button>
</form>