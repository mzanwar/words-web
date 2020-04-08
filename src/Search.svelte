<script>
  let video_list = []
  let total_hits = 0

  let index = 0
  let video_id = ""
  let start = 0
  let text = ""

  $: if (video_list.length > 0) {
	  video_id = video_list[index]['_source']['video_id'];
	  start = Math.floor(video_list[index]['_source']['start']);
	  text = video_list[index]['_source']['text'];
  }

  function search() {
    var term = document.getElementById("search-box").value;
    let normalized_term = term.replace(" ", "+");
    console.log(normalized_term);
    if (normalized_term != "") {
      query_elasticsearch(normalized_term);
    }
  }

  function next() {
	  if (index < total_hits) {
		  index += 1
	  }
  }

    function back() {
	  if (index > 0) {
		  index -= 1
	  }
  }

  function query_elasticsearch(query) {
    const Http = new XMLHttpRequest();
    const url = "http://localhost:9200/_search?q=" + query + "&size=100";
    Http.open("GET", url);
    Http.send();

    Http.onreadystatechange = function(){
      if (this.readyState == 4 && this.status == 200) {
		console.log(Http.responseText);
        total_hits = JSON.parse(Http.response)['hits']['total'];
        if (total_hits > 0) {
		  var hits = JSON.parse(Http.response)["hits"]["hits"];
		  video_list = hits
		  index = 0
        }
      } else {
		  text = "Could not find word"
	  }
    };
  }

window.addEventListener('keyup', function (e) {
    if (e.keyCode === 13) {
		search();
    }
}, false);
</script>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 2em;
    font-weight: 100;
  }

 h2 {
    color: black;
    text-transform: uppercase;
    font-size: 1.5em;
    font-weight: 200;
  }

button {
    color: black;
    text-transform: uppercase;
    font-size: 1.5em;
    font-weight: 100;
  }

  input {
	  /* height: 200px; */
	  width: 300px;
	  font-size: 1.5em;
	  text-align: center;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>

<main>
  <h1>"{text}"</h1>
  <button on:click={back}>Back</button>
  <input id="search-box" on placeholder="Search" />
  <!-- <button on:click={search}>Search</button> -->
  <button on:click={next}>Next</button>

  <h2>{index + 1} of {total_hits}</h2>
  <iframe
    title="video"
    width="600"
    height="505"
    src="https://www.youtube.com/embed/{video_id}?start={start}&autoplay=1" />
</main>
