
script
```typescript
const { createClient } = require("@sanity/client");
const fs = require("fs");
const csv = require("csv-parser");

const client = createClient({
  projectId: "uqxwe2qj",
  dataset: "staging",
  useCdn: false,
  apiVersion: "2021-10-21",
  token:
    "sk1XkRwOnlGeoYpOvPDLtXhGrA8x8WvykQhMdM5xmEUeGmIei9tw70mCFMCbvXdpTjQcttsO82UFMZXg4rk2EnlxcmnP3U5YqUCWmYxP2gE2DgFcoMwhmnUjUj8hx3dRgyNN0428tGZU31iZLy6lWYgtDjm16i4TFpljmgBEWyxN07TgT2ZN",
});

// const csvFilePath = "location.csv";

// async function updateSanityMeta() {
//   try {
//     const query = '*[_type == "meta"]';
//     const metaDocs = await client.fetch(query);

//     const csvData = new Map();

//     await new Promise((resolve, reject) => {
//       fs.createReadStream(csvFilePath)
//         .pipe(csv())
//         .on("data", (row) => {
//           const cleanUrl = row.URL.replace("https://wework.co.in", "");

//           csvData.set(cleanUrl, {
//             title: row["New Title"],
//             description: row["New Description"],
//           });
//         })
//         .on("end", resolve)
//         .on("error", reject);
//     });

//     for (const doc of metaDocs) {
//       if (doc.url && csvData.has(doc.url)) {
//         const csvEntry = csvData.get(doc.url);
//         if (doc.url === "/bangalore/infantry-road/prestige-central/") {
//           console.log("🚀 ~ updateSanityMeta ~ csvEntry:", csvEntry);
//           console.log("🚀 ~ updateSanityMeta ~ doc:", doc);
//           await client
//             .patch(doc._id)
//             .set({
//               metaTitle: csvEntry.title,
//               description: csvEntry.description,
//             })
//             .commit()
//             .then((updatedDoc) => {
//               console.log(updatedDoc, "updated");
//             })
//             .catch((err) => {
//               console.log("error", err);
//             });
//         }
//       }
//     }

//     console.log("Update process completed!");
//   } catch (error) {
//     console.error("Error:", error.message);
//   }
// }

// New function to update micromarket meta with schema information
// async function updateMicromarketSchemas(schemaFilePath) {
//   try {
//     // Fetch all meta documents from Sanity
//     const query = '*[_type == "meta"]';
//     const metaDocs = await client.fetch(query);

//     // Create a map to store schema data by URL
//     const schemaData = new Map();

//     // Read schema data from CSV
//     await new Promise((resolve, reject) => {
//       fs.createReadStream(schemaFilePath)
//         .pipe(csv())
//         .on("data", (row) => {
//           if (row.URL && row["Schema Type"] && row["Schema Code"]) {
//             const cleanUrl = row.URL.replace("https://wework.co.in", "");

//             if (!schemaData.has(cleanUrl)) {
//               schemaData.set(cleanUrl, {});
//             }

//             // Store the schema by type
//             schemaData.get(cleanUrl)[row["Schema Type"]] = row["Schema Code"];
//           }
//         })
//         .on("end", resolve)
//         .on("error", reject);
//     });

//     console.log(`Loaded schema data for ${schemaData.size} URLs`);

//     // Process URLs for micromarket pages
//     const micromarketUrls = [
//       "/bangalore/whitefield/",
//       // "/bangalore/jayanagar/",
//       // "/bangalore/btm-layout/",
//       // "/bangalore/hsr-layout/",
//     ];

//     let updatedCount = 0;

//     // Find and update meta documents for each micromarket URL
//     for (const micromarketUrl of micromarketUrls) {
//       // Find meta document for this URL
//       const metaDoc = metaDocs.find(
//         (doc) => doc.url === `coworking-space/${micromarketUrl}`
//       );

//       // If we don't have a meta document for this URL, create one
//       if (!metaDoc) {
//         console.log(
//           `No meta document found for ${micromarketUrl}, creating new one`
//         );

//         // Extract city and micromarket from URL
//         const urlParts = micromarketUrl.split("/").filter((part) => part);
//         // if (urlParts.length < 2) {
//         //   console.log(`Invalid URL format: ${micromarketUrl}`);
//         //   continue;
//         // }

//         const cityHandle = urlParts[1];
//         const micromarketHandle = urlParts[2];

//         // Create new meta document if schema data exists
//         if (schemaData.has(micromarketUrl)) {
//           const schemas = schemaData.get(micromarketUrl);

//           const newMetaDoc = {
//             _type: "meta",
//             name: micromarketHandle,
//             url: micromarketUrl,
//             metaTitle: `WeWork ${cityHandle} ${micromarketHandle} | Office Space For Rent`,
//             description: `Find the perfect workspace at WeWork ${cityHandle} ${micromarketHandle}. Book private offices, hot desks, meeting rooms & more.`,
//             seoSchemas: {
//               webPage: schemas.WebPage || null,
//               organization: schemas.organisational || null,
//               faq: schemas.FAQs || null,
//             },
//           };

//           await client
//             .create(newMetaDoc)
//             .then(() => {
//               console.log(`Created new meta document for ${micromarketUrl}`);
//               updatedCount++;
//             })
//             .catch((err) => {
//               console.error(`Error creating meta for ${micromarketUrl}:`, err);
//             });
//         }
//       }
//       // Update existing meta document
//       else if (schemaData.has(micromarketUrl)) {
//         const schemas = schemaData.get(micromarketUrl);

//         await client
//           .patch(metaDoc._id)
//           .set({
//             seoSchemas: {
//               webPage: schemas.WebPage || null,
//               organization: schemas.organisational || null,
//               faq: schemas.FAQs || null,
//             },
//           })
//           .commit()
//           .then(() => {
//             console.log(`Updated schema for ${micromarketUrl}`);
//             updatedCount++;
//           })
//           .catch((err) => {
//             console.error(`Error updating schema for ${micromarketUrl}:`, err);
//           });
//       }
//     }

//     console.log(
//       `Schema update process completed! Updated ${updatedCount} documents.`
//     );
//   } catch (error) {
//     console.error("Error:", error.message);
//   }
// }

// Function to generate schema markup for a specific micromarket
function generateSchemaMarkup(cityName, micromarketName, fullUrl) {
  return [
    {
      _key: `webPage_${cityName}_${micromarketName}`,
      type: "WebPage",
      _type: "webPageType",
      name: "Meta",
      publisher: {
        name: `WeWork India ${micromarketName}`,
        id: `${fullUrl}#WebPage`,
        type: "Organization",
      },
      id: `${fullUrl}#WebPage`,
    },
    {
      url: fullUrl,
      sameAs: [
        "https://www.facebook.com/WeWorkIndia/?brand_redir=330321486979917",
        "https://twitter.com/WeWorkIndia",
        "https://www.instagram.com/wework.in/",
        "https://www.youtube.com/@WeWorkIndia",
      ],
      _type: "organization",
      name: "WeWork India",
      logo: {
        _type: "image",
        asset: {
          _ref: "image-dbc97732f7acd87fae8b4e4916f54111d283a2cd-274x55-png",
          _type: "reference",
        },
      },
      _key: `org_${cityName}_${micromarketName}`,
      type: "Organization",
    },
    {
      mainEntity: [
        {
          type: "Question",
          acceptedAnswer: {
            text: "WeWork provides flexible and scalable solutions for businesses of all sizes:\n\nOffice Spaces:\n\nPrivate Offices: Ready-to-move-in or customisable spaces for teams of any size.\nManaged Offices: End-to-end office solutions sourced, built, and operated by WeWork.\nCoworking Spaces:\n\nAll Access Plus: Monthly membership with access to 450+ global locations.\nAll Access Pay-Per-Use: Flexible coworking for hybrid teams, billed monthly.\nOn-demand day passes: Book day passes at 40+ coworking spaces on-the-go and pay per use\nOn-demand meeting rooms: Book fully-equipped meeting rooms on-the-go and pay by the hour\nWeWork Labs: Investments, acceleration, and subsidised workspaces for startups\nAdditional Solutions:\n\nWorkplace: One platform for workspace management and employee experience\nVirtual Office by WeWork: Give your business a premium address for GST and company registration\nEvent and shoots: Host your shoots, pop-ups, and events at WeWork\nStudios: Dedicated studio space to record podcasts or shoot with green screens\nAdvertise at WeWork: Advertise your brand in our 60 locations and reach a diverse audience\nWeWork Business Solutions: A full suite of business services across HR, IT, admin, finance, and more",
            type: "Answer",
          },
          _type: "question",
          name: "What types of workspace solutions does WeWork offer?",
          _key: `faq1_${cityName}_${micromarketName}`,
        },
        {
          type: "Question",
          acceptedAnswer: {
            text: `A WeWork office space in ${cityName} offers benefits like prime Grade-A locations, modern infrastructure and amenities, flexible lease options, a thriving business community, events, community team support, mail and package handling, and sustainable workspace solutions.`,
            type: "Answer",
          },
          _type: "question",
          name: `What are the benefits of renting a WeWork office space in ${cityName}?`,
          _key: `faq2_${cityName}_${micromarketName}`,
        },
        {
          type: "Question",
          acceptedAnswer: {
            text: "To book a WeWork coworking space, visit the On-Demand page to reserve a day pass for your preferred location. For a monthly coworking membership, visit to the All Access Plus page and get in touch with us by filling out the form.",
            type: "Answer",
          },
          _type: "question",
          name: "How can I book a WeWork coworking space?",
          _key: `faq3_${cityName}_${micromarketName}`,
        },
        {
          type: "Question",
          acceptedAnswer: {
            text: "If you have a Private Office at a WeWork location, you enjoy 24/7 access to that space. To access WeWork locations, other members need to book a meeting room or workspace in advance. The Community team is on-site at each location on weekdays from 9 AM to 6 PM (apart from holidays).",
            type: "Answer",
          },
          _type: "question",
          name: "When can I access a WeWork location? What are the open hours?",
          _key: `faq4_${cityName}_${micromarketName}`,
        },
        {
          type: "Question",
          acceptedAnswer: {
            text: "Guests are welcome, provided you have booked a meeting room or have a private office. The guest count must match the room or office capacity, including the host. Guests may access the lounge area for up to 20 minutes; for visits over 20 minutes, please book a conference room using credits.",
            type: "Answer",
          },
          _type: "question",
          name: "What is WeWork's guest policy?",
          _key: `faq5_${cityName}_${micromarketName}`,
        },
      ],
      _type: "faqPageType",
      _key: `faq_${cityName}_${micromarketName}`,
      type: "FAQPage",
    },
  ];
}

// Main function to update schemas for all micromarkets
async function updateSchemaForMicromarkets() {
  try {
    // Fetch all meta documents from Sanity
    const query = '*[_type == "meta"]';
    const metaDocs = await client.fetch(query);

    // Define micromarket data - you can expand this list as needed
    const micromarks = [
      {
        city: "bangalore",
        micromarket: "cunningham-road",
        cityName: "Bangalore",
        micromarketName: "Cunningham Road",
      },
      //   {
      //     city: "bangalore",
      //     micromarket: "jayanagar",
      //     cityName: "Bangalore",
      //     micromarketName: "Jayanagar",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "btm-layout",
      //     cityName: "Bangalore",
      //     micromarketName: "BTM Layout",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "hsr-layout",
      //     cityName: "Bangalore",
      //     micromarketName: "HSR Layout",
      //   },
      {
        city: "bangalore",
        micromarket: "whitefield",
        cityName: "Bangalore",
        micromarketName: "Whitefield",
      },
      //   {
      //     city: "bangalore",
      //     micromarket: "jp-nagar",
      //     cityName: "Bangalore",
      //     micromarketName: "JP Nagar",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "outer-ring-road",
      //     cityName: "Bangalore",
      //     micromarketName: "Outer Ring Road",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "marathahalli",
      //     cityName: "Bangalore",
      //     micromarketName: "Marathahalli",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "kalyan-nagar",
      //     cityName: "Bangalore",
      //     micromarketName: "Kalyan Nagar",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "malleshwaram",
      //     cityName: "Bangalore",
      //     micromarketName: "Malleshwaram",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "koramangala",
      //     cityName: "Bangalore",
      //     micromarketName: "Koramangala",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "mg-road",
      //     cityName: "Bangalore",
      //     micromarketName: "MG Road",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "bannerghatta-main-rd",
      //     cityName: "Bangalore",
      //     micromarketName: "Bannerghatta Main Road",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "infantry-road",
      //     cityName: "Bangalore",
      //     micromarketName: "Infantry Road",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "jp-nagar-7th-phase",
      //     cityName: "Bangalore",
      //     micromarketName: "JP Nagar 7th Phase",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "marathahalli-sarjapur-outer-ring-road",
      //     cityName: "Bangalore",
      //     micromarketName: "Marathahalli Sarjapur Outer Ring Road",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "indiranagar",
      //     cityName: "Bangalore",
      //     micromarketName: "Indiranagar",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "domlur",
      //     cityName: "Bangalore",
      //     micromarketName: "Domlur",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "bellandur",
      //     cityName: "Bangalore",
      //     micromarketName: "Bellandur",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "hebbal",
      //     cityName: "Bangalore",
      //     micromarketName: "Hebbal",
      //   },
      //   {
      //     city: "bangalore",
      //     micromarket: "mahadevapura",
      //     cityName: "Bangalore",
      //     micromarketName: "Mahadevapura",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "kandivali",
      //     cityName: "Mumbai",
      //     micromarketName: "Kandivali",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "kandivali-east",
      //     cityName: "Mumbai",
      //     micromarketName: "Kandivali East",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "lower-parel",
      //     cityName: "Mumbai",
      //     micromarketName: "Lower Parel",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "nariman-point",
      //     cityName: "Mumbai",
      //     micromarketName: "Nariman Point",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "goregaon-west",
      //     cityName: "Mumbai",
      //     micromarketName: "Goregaon West",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "mira-road",
      //     cityName: "Mumbai",
      //     micromarketName: "Mira Road",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "dadar",
      //     cityName: "Mumbai",
      //     micromarketName: "Dadar",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "goregaon",
      //     cityName: "Mumbai",
      //     micromarketName: "Goregaon",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "vikhroli",
      //     cityName: "Mumbai",
      //     micromarketName: "Vikhroli",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "thane",
      //     cityName: "Mumbai",
      //     micromarketName: "Thane",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "andheri-east",
      //     cityName: "Mumbai",
      //     micromarketName: "Andheri East",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "borivali-east",
      //     cityName: "Mumbai",
      //     micromarketName: "Borivali East",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "borivali-west",
      //     cityName: "Mumbai",
      //     micromarketName: "Borivali West",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "borivali",
      //     cityName: "Mumbai",
      //     micromarketName: "Borivali",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "south-mumbai",
      //     cityName: "Mumbai",
      //     micromarketName: "South Mumbai",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "mulund",
      //     cityName: "Mumbai",
      //     micromarketName: "Mulund",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "andheri-west",
      //     cityName: "Mumbai",
      //     micromarketName: "Andheri West",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "vile-parle",
      //     cityName: "Mumbai",
      //     micromarketName: "Vile Parle",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "churchgate",
      //     cityName: "Mumbai",
      //     micromarketName: "Churchgate",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "malad",
      //     cityName: "Mumbai",
      //     micromarketName: "Malad",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "worli",
      //     cityName: "Mumbai",
      //     micromarketName: "Worli",
      //   },
      //   {
      //     city: "mumbai",
      //     micromarket: "bkc",
      //     cityName: "Mumbai",
      //     micromarketName: "BKC",
      //   },
      //   {
      //     city: "gurgaon",
      //     micromarket: "sector-44",
      //     cityName: "Gurgaon",
      //     micromarketName: "Sector 44",
      //   },
      //   {
      //     city: "gurgaon",
      //     micromarket: "mg-road-gurgaon",
      //     cityName: "Gurgaon",
      //     micromarketName: "MG Road",
      //   },
      //   {
      //     city: "gurgaon",
      //     micromarket: "cybercity",
      //     cityName: "Gurgaon",
      //     micromarketName: "Cybercity",
      //   },
      //   {
      //     city: "gurgaon",
      //     micromarket: "bristol-chowk",
      //     cityName: "Gurgaon",
      //     micromarketName: "Bristol Chowk",
      //   },
      //   {
      //     city: "gurgaon",
      //     micromarket: "golf-course-road",
      //     cityName: "Gurgaon",
      //     micromarketName: "Golf Course Road",
      //   },
      //   {
      //     city: "gurgaon",
      //     micromarket: "udyog-vihar",
      //     cityName: "Gurgaon",
      //     micromarketName: "Udyog Vihar",
      //   },
      //   {
      //     city: "pune",
      //     micromarket: "viman-nagar",
      //     cityName: "Pune",
      //     micromarketName: "Viman Nagar",
      //   },
      //   {
      //     city: "pune",
      //     micromarket: "baner",
      //     cityName: "Pune",
      //     micromarketName: "Baner",
      //   },
      //   {
      //     city: "pune",
      //     micromarket: "kalyani-nagar",
      //     cityName: "Pune",
      //     micromarketName: "Kalyani Nagar",
      //   },
      //   {
      //     city: "pune",
      //     micromarket: "koregaon-park",
      //     cityName: "Pune",
      //     micromarketName: "Koregaon Park",
      //   },
      //   {
      //     city: "pune",
      //     micromarket: "kharadi",
      //     cityName: "Pune",
      //     micromarketName: "Kharadi",
      //   },
      //   {
      //     city: "pune",
      //     micromarket: "pimpri-chinchwad",
      //     cityName: "Pune",
      //     micromarketName: "Pimpri Chinchwad",
      //   },
      //   {
      //     city: "pune",
      //     micromarket: "hadapsar",
      //     cityName: "Pune",
      //     micromarketName: "Hadapsar",
      //   },
      //   {
      //     city: "pune",
      //     micromarket: "magarpatta",
      //     cityName: "Pune",
      //     micromarketName: "Magarpatta",
      //   },
      //   {
      //     city: "hyderabad",
      //     micromarket: "hitec-city",
      //     cityName: "Hyderabad",
      //     micromarketName: "Hitec City",
      //   },
      //   {
      //     city: "hyderabad",
      //     micromarket: "madhapur",
      //     cityName: "Hyderabad",
      //     micromarketName: "Madhapur",
      //   },
      //   {
      //     city: "hyderabad",
      //     micromarket: "gachibowli",
      //     cityName: "Hyderabad",
      //     micromarketName: "Gachibowli",
      //   },
      //   {
      //     city: "hyderabad",
      //     micromarket: "financial-district",
      //     cityName: "Hyderabad",
      //     micromarketName: "Financial District",
      //   },
      //   {
      //     city: "hyderabad",
      //     micromarket: "kukatpally",
      //     cityName: "Hyderabad",
      //     micromarketName: "Kukatpally",
      //   },
      //   {
      //     city: "hyderabad",
      //     micromarket: "kondapur",
      //     cityName: "Hyderabad",
      //     micromarketName: "Kondapur",
      //   },
      //   {
      //     city: "noida",
      //     micromarket: "sector-125",
      //     cityName: "Noida",
      //     micromarketName: "Sector 125",
      //   },
      //   {
      //     city: "noida",
      //     micromarket: "greater-noida",
      //     cityName: "Noida",
      //     micromarketName: "Greater Noida",
      //   },
      //   {
      //     city: "noida",
      //     micromarket: "sector-2",
      //     cityName: "Noida",
      //     micromarketName: "Sector 2",
      //   },
      //   {
      //     city: "noida",
      //     micromarket: "sector-63",
      //     cityName: "Noida",
      //     micromarketName: "Sector 63",
      //   },
      //   {
      //     city: "noida",
      //     micromarket: "sector-18",
      //     cityName: "Noida",
      //     micromarketName: "Sector 18",
      //   },
      //   {
      //     city: "noida",
      //     micromarket: "sector-62",
      //     cityName: "Noida",
      //     micromarketName: "Sector 62",
      //   },
      //   {
      //     city: "noida",
      //     micromarket: "sector-16",
      //     cityName: "Noida",
      //     micromarketName: "Sector 16",
      //   },
      //   {
      //     city: "chennai",
      //     micromarket: "guindy",
      //     cityName: "Chennai",
      //     micromarketName: "Guindy",
      //   },
      //   {
      //     city: "chennai",
      //     micromarket: "manapakkam",
      //     cityName: "Chennai",
      //     micromarketName: "Manapakkam",
      //   },
      //   {
      //     city: "delhi",
      //     micromarket: "malviya-nagar",
      //     cityName: "Delhi",
      //     micromarketName: "Malviya Nagar",
      //   },
      //   {
      //     city: "delhi",
      //     micromarket: "saket",
      //     cityName: "Delhi",
      //     micromarketName: "Saket",
      //   },
    ];

    let updatedCount = 0;
    let createdCount = 0;

    // Process each micromarket
    for (const market of micromarks) {
      // Construct the URL patterns to check
      const urlPatterns = [
        `/coworking-space/${market.city}/${market.micromarket}/`,
        // `/${market.city}/${market.micromarket}/`,
      ];

      // Find meta document for this micromarket
      const metaDoc = metaDocs.find((doc) => urlPatterns.includes(doc.url));

      // Full URL for schema
      const fullUrl = `https://wework.co.in/${market.city}/${market.micromarket}/`;

      // Generate schema markup
      const schemaMarkup = generateSchemaMarkup(
        market.cityName,
        market.micromarketName,
        fullUrl
      );

      if (metaDoc) {
        // Update existing meta document with schema
        try {
          await client
            .patch(metaDoc._id)
            .set({
              schemaData: {
                schemaMarkup,
              },
            })
            .commit();
          console.log(
            `Updated schema for ${market.cityName} ${market.micromarketName}`
          );
          updatedCount++;
        } catch (err) {
          console.error(
            `Error updating schema for ${market.cityName} ${market.micromarketName}: ${err.message}`
          );
        }
      } else {
        // Create new meta document with schema
        const newMetaDoc = {
          _type: "meta",
          name: market.micromarket,
          url: urlPatterns[0], // Use coworking-space URL pattern
          metaTitle: `WeWork ${market.cityName} ${market.micromarketName} | Office Space For Rent`,
          description: `Find the perfect workspace at WeWork ${market.cityName} ${market.micromarketName}. Book private offices, hot desks, meeting rooms & more.`,
          schemaData: {
            schemaMarkup,
          },
        };

        try {
          await client.create(newMetaDoc);
          console.log(
            `Created new meta document for ${market.cityName} ${market.micromarketName}`
          );
          createdCount++;
        } catch (err) {
          console.error(
            `Error creating meta for ${market.cityName} ${market.micromarketName}: ${err.message}`
          );
        }
      }
    }

    return { updated: updatedCount, created: createdCount };
  } catch (error) {
    throw new Error(error.message);
  }
}

// When running directly from command line
if (require.main === module) {
  updateSchemaForMicromarkets()
    .then((result) => {
      console.log(
        `Schema update completed. Updated: ${result.updated}, Created: ${result.created}`
      );
    })
    .catch((error) => {
      console.error(`Error: ${error.message}`);
    });
}

// Export the function
module.exports = {
  updateSchemaForMicromarkets,
};


```