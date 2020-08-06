---
title: Utiliser les FormRequest dans une API Restful
date: 2020-08-06T02:00:46.835Z
image: /images/audit-3737447_1920.jpg
tags:
  - laravel
  - api
  - rest
  - restful
  - formrequest
  - validation
draft: false
---
Lorsque vient le temps, dans Laravel, de valider du contenu soumis par un utilisateur, les [Form Requests](https://laravel.com/docs/master/validation#form-request-validation) se révèlent d'une efficacité sans pareil.

Ils permettent d'isoler toute la logique de validation des données dans une classe dédiée, et de bloquer la requête si les données sont invalides ou encore si l'utilisateur n'aurait pas l'autorisation nécessaire.

Si jamais dans votre API, les FormRequest renvoient ne renvoie pas le résultat escompté lorsque les données ne sont pas valides, c'est probablement car la requête ne spécifie pas qu'elle attend un résultat en JSON. Du coup, Laravel renvoie plutôt une redirection avec les messages d'erreurs.
<!-- excerpt -->
La première solution serait de toujours envoyer, dans la requête, un entête pour expliciter qu'on attend un JSON en réponse :

```http
Accept: aplication/json
```

La seconde solution que je vous propose est de créer une classe qui hérite de FormRequest, on l'appellera ApiFormRequest et c'est elle qui aura la responsabilité de renvoyer les erreurs au format JSON. Cette classe se logera très bien dans le dossier App/Http/Requests

```php
<?php


namespace App\Http\Requests;


use Illuminate\Contracts\Validation\Validator;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Http\Exceptions\HttpResponseException;
use Illuminate\Http\JsonResponse;
use Illuminate\Validation\ValidationException;

class ApiFormRequest extends FormRequest
{
    /**
     * Handle a failed validation attempt properly for API.
     *
     * @param  \Illuminate\Contracts\Validation\Validator  $validator
     * @return void
     *
     * @throws \Illuminate\Validation\ValidationException
     */
    protected function failedValidation(Validator $validator)
    {
        $errors = (new ValidationException($validator))->errors();

        throw new HttpResponseException(
            response()->json([
                "errors" => $errors
            ], JsonResponse::HTTP_UNPROCESSABLE_ENTITY)
        );
    }

}
```

Explications : 

On passe ``$validator`` à la classe ``ValidationException`` et on récupère les messages d'erreur de validation.
Ensuite on déclenche une exception HTTP qui se chargera de transmettre les erreurs à l'API en ajoutant un statut de réponse correspondant à la valeur de ``HTTP_UNPROCESSABLE_ENTITY``.

Il ne reste plus qu'à faire hériter nos FormRequest de cette nouvelle classe.

<hr>

<small>Image par <a href="https://pixabay.com/fr/users/mohamed_hassan-5229782/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3737447">Mohamed Hassan</a> de <a href="https://pixabay.com/fr/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3737447">Pixabay</a></small>