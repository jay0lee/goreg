<html>
<head>
<title>GCP Organization Restriction Extension Generator</title>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
<script src="jszip.min.js"></script>
<script src="FileSaver.min.js"></script>
<script>
function handleClick() {
  $.getJSON('manifest_template.json', function(manifest) {
    $.getJSON('rule_template.json', function(rule) {
      var ext_name = document.getElementById('ext_name').value;
      var org_ids = document.getElementById('org_ids').value.split('\n');
      for(let i = 0; i < org_ids.length; i++) {
          org_ids[i] = "organizations/" + org_ids[i];
          }
      manifest.name = ext_name;
      var nowd = new Date();
      var year = nowd.getUTCFullYear().toString();
      var month = (nowd.getUTCMonth() + 1).toString().padStart(2, "0");
      var dom = nowd.getUTCDate().toString().padStart(2, "0");
      var hour = nowd.getUTCHours().toString().padStart(2, "0");
      var minutes = nowd.getUTCMinutes().toString().padStart(2, "0");
      var seconds = nowd.getUTCSeconds().toString().padStart(2, "0");
      var ver_str = `${year}.${month}${dom}.${hour}.${minutes}${seconds}`;
      manifest.version = ver_str;
      var raw_header = {"resources": org_ids,
                        "options": "strict"};
      var raw_header_str = JSON.stringify(raw_header);
      var encoded_header = btoa(raw_header_str).replace('+', '-').replace('/', '_').replace(/=+$/, '');
      rule[0].action.requestHeaders[0].value = encoded_header;
      var zip = new JSZip();
      zip.file("rules1.json", JSON.stringify(rule, null, 2));
      zip.file("manifest.json", JSON.stringify(manifest, null, 2));
      zip.generateAsync({type:"blob"}).then(function(content) {
        saveAs(content, "org-restriction-" + manifest.version + ".zip");
      });
    });
  });
}

</script>
</head>
<body>
  <h2>GCP Organization Restriction Extension Generator</h2>
  This website will generate a Google Chrome extension which will restrict your the Google Cloud Platform organizations your browser is allowed to access. It does this by setting the X-Goog-Allowed-Resources header on any requests made to GCP domains. See Google's documentation on <a href="https://cloud.google.com/resource-manager/docs/organization-restrictions/overview">Organization Restrictions</a> for more information on the capability.
  
<form name="exdetails" method="post" onSubmit="handleClick(); return false">
        Name your extension: <input type="text" id="ext_name" name="ext_name"><br>
        List GCP Organizations IDs to allow (number only) :<br>
        <textarea id="org_ids" name="org_ids" rows="25" cols="20"></textarea><br>
        <br>
        <input name="Submit"  type="submit" value="Generate Extension" />
</form>

  <h3>Quick test for your extension</h3>
  <ol>
    <li>Download and unzip your generated extension.</li>
  <li><a href="https://developer.chrome.com/docs/extensions/mv3/getstarted/development-basics/#load-unpacked">Install the unpacked extension</a></li>
  <li>You should now be restricted to using the GCP organizations you allowlisted when generating your extension.</li>
  </ol>
  
    <h3>Force installing the extension for users.</h3>
  <ol>
    <li>Download the extension and leave it as a .zip file.</li>
    <li>Upload the extension to the <a href="https://chrome.google.com/webstore/devconsole/">Chrome Web Store Developer Dashboard</a>.</li>
    <li>Publish the extension to your domain.</li>
    <li>In the admin console, force install the extension for your users.</li>
    <li>If you need to modify the list of allowlisted GCP organizations, generate a new extension with the list and then upload the new version to the CWS developer dashboard.</li>
  </ol>

</body>
