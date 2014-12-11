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
```scala
// Før directives
case req@GET(ContextPath(_, urls.boliglanListe())) =>
      val rikerProdukter: List[RiktLanProdukt] = lanService.aktiveBoliglanProdukter
      sporingsService.registrerProduktvisninger(rikerProdukter.map(rp ⇒ (rp.produkt, rp.rangeringsdata)))
      val json = rikerProdukter.asJson
      Ok ~> JsonContent ~> ResponseString(json.spaces4)

// Etter directives      
case ContextPath(_, urls.boliglanListe()) =>
      directive.registrerProduktvisninger(Boliglan)      
      
private object directive {
  def registrerProduktvisninger(produktType: ProduktType) =
    for {
      _             ← GET
      cookie        ← data.as.Option[String] named "USERID"
      prisantydning ← data.as.Option[Long] named "prisantydning"
      postnr        ← data.as.Option[String] named "postnummer"
      rikeProdukter ← directive.hentAktiveProdukter(produktType, postnr)
    } yield {
      sporingsService.registrerProduktvisninger(
        rikeProdukter.map(rp ⇒ (rp.produkt, rp.rangeringsdata)),
        cookie,
        prisantydning)
      val json = (Option(Referansenummer.generer), rikeProdukter).asJson.spaces4
      Ok ~> JsonContent ~> ResponseString(json)
    }
}
```

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


