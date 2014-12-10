Scala Arkitekturdag
====================

Penger.no

Teamet
------
Øyvind
Nina
Eirik
Jari
Torbjørn
Jørgen
Stein Kåre

Historien
---------

2009 -> Jørgen forteller om den golden days - da man trodde alt var bra!

- Spring MVC
- Hibernate
- Java

I DAG
=====

Overordenet
-----------

### Forskjellige produkter

- Boliglån
- Forsikring
- Billån
- Strøm
- ...

### Sepearsjon mellom Front og Backend

Fordeler

- Spesialisere 

Ulemper
- Involverte i en feature

#### Frontend for brukere/søkere

- Node.js
- Client.js

#### Frontend for Admin

- Rendrer html via Scala
  - Slick CRUD

### Backend

- HTTP 
  - Unfiltered m/Directives
  - (Cyber)Suit
- Middelware
  - Scala
    - Cake
  - Dispatch
  - Argonaut
  - Scalaz
- Backend
  - Slick
  - Elastic4s

## Unfiltered


### Directives

## CyberSuite

## Cake

## Dispatch

### WS/SOAP

## Argonaut
> Purely Functional JSON in Scala

Vi bruker Argonaut for konvertering til/fra JSON

```scala
// Full smack import
import argonaut.Argonaut._
import argonaut._

// Codec for case class goodness
case class Person(name: String, age: Int, things: List[String])
implicit def PersonCodecJson =
  casecodec3(Person.apply, Person.unapply)("name", "age", "things")

// Hjemmebakt codec
implicit val BigDecimalCodecJson: CodecJson[BigDecimal] =
    CodecJson(
      bd ⇒ jString(bd.toString()),
      c ⇒ c.as[String].map(s ⇒ BigDecimal(s))
    )

// Encoding json mer manuelt
implicit def RedirectUrlEncodeJson: EncodeJson[RedirectUrl] =
    EncodeJson((ru: RedirectUrl) ⇒
      ("apiUrl" := ru.apiUrl) ->: ("browserUrl" := ru.browserUrl) ->: jEmptyObject)

// Decoding kan også gjøres mer custom
implicit val produktivisningDecoder: DecodeJson[Produktvisning] = {
    DecodeJson(c ⇒ for {
      produkttype ← (c --\ Produktvisning.produkttype).as[String].map{ProduktType(_)}
      produktid   ← (c --\ Produktvisning.produktid).as[Long].map { pi ⇒
        produkttype match {
          case (Billan | Boliglan)  ⇒ LanProduktId(pi)
          case (Bilforsikring)      ⇒ ForsikringProduktId(pi)
        }
      }
    } yield {
      Produktvisning(produktid, produkttype)
    })
  }
 
// Med impicits i scope får man json-objekt-generering der man trenger det 
val json = service.hentPerson(navn).asJson.spaces4
Ok ~> JsonContent ~> ResponseString(json)
```

## Scalaz

## Slick

## Elastic4s


