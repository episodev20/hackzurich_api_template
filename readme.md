app.py is a template for the API in lambda
is short and selfexplanatory

openvirome.html is the list of JS scripts injected in openvirome.com squarespace code sections

first script with SHA1 function is just a hashing implementation used to match the filenames posted by aws lambda to S3 
without getting response from lambda and still knwoing what file to fetch.
(it was a temporary solution until we get the responses from lambda, now they are availible, so it has to be redone)

Second script uses link parameters after ? as filenames for html files in S3 aws
and uses them instead of base example files in iframe
```
<script>
    let url = window.location.href;
    var arr = url.split('?');
    if (url.length > 1 && arr[1] !== undefined) {
      var newSrc= "https://s3.amazonaws.com/openvirome.com/" + arr[1] + ".html"
      document.getElementById("iframe").src=newSrc;
      document.getElementById('iframe').contentWindow.location.reload()
    }
    </script>
```

Following Postdata script is just a curl Post request implementation

next src_update function is triggered by "send" button (below)
and get the sequence inputed by user
checks that it is correct:
1. removes everything but letters, and makes to_upper)
2. if no header text is present "OpenVirome" is used
3. everything before firs > and after last > (if present) is truncated

after that a sequence is used to make a Postdata call to lambda API and get
the link to the report. (if the report is not ready yet, lambda starts generation of it)
the link is then inputed in Iframe and Iframe is reloaded

```
<script>
    function src_update() {
      
    sequence_text = document.getElementById("sequence_text").value
        
    // cut off everything before first > and after second
    // if second present
    sequence_text = sequence_text.split('>')[1]
    // split by new lines
    sequence_text = sequence_text.split('\n')
        
    // first element is header    
    header_text = ">" + sequence_text[0]
        
    // rest is protein sequence
    protein_text = sequence_text.slice(1, )
    protein_text = protein_text.join()
    
    if (header_text == undefined){
      header_text = ">OpenVirome"
    }
    
    // in protein text can be only letters and they have
    // to be to_upper()
    protein_text = protein_text.replace(/[^A-Za-z]/g, '').toUpperCase();
        
    sequence_text_right = header_text + "\n" + protein_text
    // document.getElementById("text_output").value = sequence_text_right
    
    var newSrc="placeholder_for_response"

    var data_call = { ["sequence"]: sequence_text_right };
    postData('https://3niuza5za3.execute-api.us-east-1.amazonaws.com/default/api-lambda', data_call)
      .then(data => {
        newSrc = "https://s3.amazonaws.com/openvirome.com/" + SHA1(sequence_text_right) + ".html";
        document.getElementById("iframe").src=newSrc;
        document.getElementById("download_button").href=newSrc;
        document.getElementById('iframe').contentWindow.location.reload()}
      );   
    }		
    </script>
```

Input form where we get user input (classes are taken from squarspace code)

```
    <form>
                  <div id="textarea-2230f9d7-d64e-41bc-a3dd-e79bc8c10dc8" class="form-item field textarea required">
                <label class="title" for="textarea-2230f9d7-d64e-41bc-a3dd-e79bc8c10dc8-field">
                  Sequence, in FASTA format
                    <span class="required" aria-hidden="true">*</span>
                </label>
                    <textarea id="sequence_text" class="field-element" id="textarea-2230f9d7-d64e-41bc-a3dd-e79bc8c10dc8-field" placeholder="Enter your sequence" aria-required="true"></textarea>
                  </div>
            </div>
```
          
    
          
Send button with classes from squarespace and connected src_update function
    

```   
    <div data-animation-role="button" class="
              form-button-wrapper
              
                form-button-wrapper--align-left
              
            ">
            <input class="button sqs-system-button sqs-editable-button" type="button" onclick="src_update()" value="Send">
          </div>
          ```