# indexing.json
https://primebenefithub.blogspot.com/indexing.json

<?php

$url = $_GET['url'];

$json = 'indexing.json';

$data = json_decode(file_get_contents($json), true);

$token_url = "https://oauth2.googleapis.com/token";

$post_data = [
"grant_type" => "urn:ietf:params:oauth:grant-type:jwt-bearer",
"assertion" => createJWT($data)
];

function createJWT($data){

$header = json_encode(['alg'=>'RS256','typ'=>'JWT']);

$time = time();

$payload = json_encode([
"iss"=>$data['client_email'],
"scope"=>"https://www.googleapis.com/auth/indexing",
"aud"=>"https://oauth2.googleapis.com/token",
"exp"=>$time+3600,
"iat"=>$time
]);

$base64URLHeader = str_replace(['+','/','='],['-','_',''],base64_encode($header));
$base64URLPayload = str_replace(['+','/','='],['-','_',''],base64_encode($payload));

$signature = '';

openssl_sign(
$base64URLHeader.".".$base64URLPayload,
$signature,
$data['private_key'],
"SHA256"
);

$base64URLSignature = str_replace(['+','/','='],['-','_',''],base64_encode($signature));

return $base64URLHeader.".".$base64URLPayload.".".$base64URLSignature;

}

$ch = curl_init($token_url);

curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($post_data));

$response = curl_exec($ch);

$token = json_decode($response, true)['access_token'];

curl_close($ch);

$api = "https://indexing.googleapis.com/v3/urlNotifications:publish";

$data = json_encode([
"url"=>$url,
"type"=>"URL_UPDATED"
]);

$ch = curl_init($api);

curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

curl_setopt($ch, CURLOPT_POSTFIELDS, $data);

curl_setopt($ch, CURLOPT_HTTPHEADER, [
"Content-Type: application/json",
"Authorization: Bearer ".$token
]);

$result = curl_exec($ch);

echo $result;

?>
