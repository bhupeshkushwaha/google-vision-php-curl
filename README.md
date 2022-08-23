# google-vision-php-curl

```
<?php
try {
    /** Get all object from image it  */
    $imagePath = __DIR__ . DIRECTORY_SEPARATOR . "IMAGE_FILE_NAME";
    $imagePathContent = file_get_contents($imagePath);
    $imagePathEncoded = base64_encode($imagePathContent);

    $curl = curl_init();
    curl_setopt_array($curl, array(
        CURLOPT_URL => 'https://vision.googleapis.com/v1/images:annotate?key=GOOGLE_VISION_KEY',
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_ENCODING => '',
        CURLOPT_MAXREDIRS => 10,
        CURLOPT_TIMEOUT => 0,
        CURLOPT_FOLLOWLOCATION => true,
        CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
        CURLOPT_CUSTOMREQUEST => 'POST',
        CURLOPT_POSTFIELDS => '{
                "requests": [
                    {
                        "image": {
                            "content": "' . $imagePathEncoded . '"
                        },
                        "features": [
                            {
                                "maxResults": 10,
                                "type": "OBJECT_LOCALIZATION"
                            },
                            {
                                "type": "LABEL_DETECTION",
                                "maxResults": 10
                            },
                            {
                                "type": "FACE_DETECTION",
                                "maxResults": 10
                            },
                            {
                                "type": "LANDMARK_DETECTION",
                                "maxResults": 10
                            },
                            {
                                "type": "LOGO_DETECTION",
                                "maxResults": 10
                            },
                            {
                                "type": "IMAGE_PROPERTIES",
                                "maxResults": 10
                            },                           
                            {
                                "type": "CROP_HINTS",
                                "maxResults": 10
                            },
                            {
                                "type": "WEB_DETECTION",
                                "maxResults": 5
                            },
                            {
                                "type": "SAFE_Search_DETECTION",
                                "maxResults": 10
                            },
                            {
                                "type": "TEXT_DETECTION",
                                "maxResults": 10
                            }
                        ]
                    }
                ]
            }',
        CURLOPT_HTTPHEADER => array(
            'Accept:  application/json',
            'Content-Type:  application/json',
            'Referer: YOUR_URL'
        ),
    ));

    $response = curl_exec($curl);
    curl_close($curl);

    $fileName = time() . ".json";
    $fp = fopen(__DIR__ . DIRECTORY_SEPARATOR . $fileName, "wb");
    fwrite($fp, $response);
    fclose($fp);

    $allData = json_decode($response);

    //$response = file_get_contents("vision-data.json");
    //$allData = json_decode($response);
    $tagList = [];

    $labelAnnotations = $allData->responses[0]->labelAnnotations ?? [];
    foreach ($labelAnnotations as $data) {
        if (isset($data->description) && !empty($data->description)) {
            array_push($tagList, $data->description);
        }
    }

    //$safeSearchAnnotation = $allData->responses[0]->safeSearchAnnotation ?? [];
    //$imagePropertiesAnnotation = $allData->responses[0]->imagePropertiesAnnotation ?? [];
    //$cropHintsAnnotation = $allData->responses[0]->cropHintsAnnotation ?? []];

    $webDetection = $allData->responses[0]->webDetection ?? [];
    foreach ($webDetection->webEntities ?? [] as $data) {
        if (isset($data->description) && !empty($data->description)) {
            array_push($tagList, $data->description);
        }
    }
    foreach ($webDetection->bestGuessLabels ?? [] as $data) {
        if (isset($data->label) && !empty($data->label)) {
            array_push($tagList, $data->label);
        }
    }

    $localizedObjectAnnotations = $allData->responses[0]->localizedObjectAnnotations ?? [];
    foreach ($localizedObjectAnnotations as $data) {
        if (isset($data->name) && !empty($data->name)) {
            array_push($tagList, $data->name);
        }
    }

    $tagList = array_map(function ($value) {
        return str_replace(' ', '-', strtolower($value));
    }, array_filter(array_unique($tagList)));

    print_r($tagList);
    exit;
} catch (Exception $e) {
    echo $e->getMessage();
}
```
