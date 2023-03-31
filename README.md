# testProductDashboard

import {faker} from "@faker-js/faker";
import {clickFromList, clickOnButton, clickOnEl, clickOnFirstFromTheList, inputText} from "../../helper/methods";
import {aliasMutation} from "../../helper/graphql-test-utils";
import {buttons, urlEndpoints} from "../../helper/names";
import {verifyTextInTable, verifyTextOfEl, verifyUrl} from "../../helper/asserts";
import {product, productModal} from "../../helper/selectors";
import {businessOwner} from "../../helper/testData";
import {baseEnvData} from "../../config/_base";

describe('Product dashboard verification', () => {
    const productFieldsRequired = {
        name: faker.commerce.productName(),
        price: faker.commerce.price(1, 99999999, 2),
        category: 'dropdownSelect()',
        vendor: 'dropdownSelect()'
    }

    const fillInProductForm = (selector, dataTest) => {
        for (let key1 in dataTest) {
            switch (key1) {
                case "category":
                case "vendor":
                case "brand": {
                    clickOnEl(selector[key1 + 'Button']);
                    clickFromList(selector[key1 + 'List']);
                    cy.get(selector[key1 + 'Button'])
                        .then(el => {
                            dataTest[key1] = el.val()
                        });
                    break;
                }
                default: {
                    inputText(selector[key1], dataTest[key1]);
                    break;
                }
            }
        }
        cy.intercept('POST', '/graphql', (req) => {
            aliasMutation(req, 'ProductCreate')
        })
        clickOnButton(buttons.save)
        cy.wait(`@gqlProductCreateMutation`).its('response.body.data.productCreate').then((value) => {
            cy.getItemApi(value.__typename, value._id).then((body) => {
                const newProduct: string[] = [
                    body.code,
                    body.name,
                    body.category,
                    body.vendor.name,
                    body.price,
                    body.cost,
                    body.brand,
                    body.brandNumber,
                    body.productCode
                ]
                newProduct.forEach(el => {
                    if (el) verifyTextOfEl('tbody tr:first', el)
                })

            })
        })
    };

    // const findElByIndex = (selector, index) => {
    //     cy.get(selector).eq(index).click()
    // }

    // const clickOnElInTable = (row: number, column: number, text) => {
    //     cy.get(`tbody tr:eq(${row - 1}) td:eq(${column - 1})`).should('be.visible').and('contain.text', text).then((el) => {
    //         cy.wrap(el).click({force: true});
    //     })
    // }

    const clickOnElFromTable = (selector, text) => {
        cy.get('tbody>tr>td').find(selector).should('be.visible').and('contain.text', text).click()
    }

    const verifyTextInTableIsClickable = (row: number, column: number, text) => {
        cy.get(`tbody tr:eq(${row - 1}) td:eq(${column - 1})`).should('be.visible').and('contain.text', text).click({});
    }

    let vendorName, vendorId: string
    before(() => {
        cy.apiLogin(baseEnvData.businessOwner.email, baseEnvData.businessOwner.password);
        cy.createItemApi('vendor').then((value) => {
            vendorName = value.name
            vendorId = value._id
        })

    })

    beforeEach(() => {
        cy.loginSession(businessOwner)
        cy.visit(urlEndpoints.productEndpoint);
        clickOnButton(product.create)
    })

    it('Verify code is a link', () =>{
        fillInProductForm(productModal, productFieldsRequired);
        // cy.get('.regularTable').contains('td', 'PROD').click({force: true})
        // cy.wait(3000)
        // verifyUrl(urlEndpoints.productEndpoint)
        // clickOnFirstFromTheList('.regularTable>tbody>tr>td>a')
       //  clickOnElInTable(1, 1, 'PROD')
        //  cy.get(`${newProduct.code}`).click()
        verifyTextInTableIsClickable(1, 1, )


