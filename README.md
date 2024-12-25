#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct stockManager {
    char reference[20];
    char name[20];
    int quantity;
    float sellingPrice;
    float purchasePrice;
    struct stockManager* next;
} stockManager;

void displayParts(stockManager* head) {
    stockManager* current = head;
    if (current == NULL) {
        printf("There are no parts.\n");
        return;
    }
    while (current != NULL) {
        printf("--------------------------------------------------------\n");
        printf("Reference: %s\n", current->reference);
        printf("Name: %s\n", current->name);
        printf("Quantity: %d\n", current->quantity);
        printf("Selling Price: %.2f DA\n", current->sellingPrice);
        printf("Purchase Price: %.2f DA\n", current->purchasePrice);
        printf("--------------------------------------------------------\n");
        current = current->next;
    }
}

void deletePart(stockManager** head, char reference[]) {
    stockManager* current = *head;
    stockManager* prev = NULL;

    while (current != NULL && strcmp(current->reference, reference) != 0) {
        prev = current;
        current = current->next;
    }

    if (current == NULL) {
        printf("Part not found.\n");
        return;
    }

    if (prev == NULL) {
        *head = current->next;
    } else {
        prev->next = current->next;
    }
    free(current);
    printf("Part deleted.\n");
}

void addPart(stockManager** head, char reference[], char name[], int quantity, float sellingPrice, float purchasePrice) {
    stockManager* newPart = (stockManager*)malloc(sizeof(stockManager));
    strcpy(newPart->reference, reference);
    strcpy(newPart->name, name);
    newPart->quantity = quantity;
    newPart->sellingPrice = sellingPrice;
    newPart->purchasePrice = purchasePrice;
    newPart->next = NULL;

    if (*head == NULL) {
        *head = newPart;
    } else {
        stockManager* current = *head;
        while (current->next != NULL) {
            current = current->next;
        }
        current->next = newPart;
    }
    printf("Part added.\n");
}

stockManager* searchParts(stockManager* head, char reference[]) {
    stockManager* current = head;
    while (current != NULL) {
        if (strcmp(current->reference, reference) == 0) {
            return current;
        }
        current = current->next;
    }
    printf("Part not found.\n");
    return NULL;
}

void saveParts(stockManager* head, const char* fileName) {
    FILE* file = fopen(fileName, "w");
    if (!file) {
        printf("Error: Unable to open file for saving.\n");
        return;
    }

    stockManager* current = head;
    while (current != NULL) {
        fprintf(file, "%s %s %d %.2f %.2f\n",
                current->reference, current->name, current->quantity,
                current->purchasePrice, current->sellingPrice);
        current = current->next;
    }
    fclose(file);
    printf("Stock saved to file.\n");
}

void loadParts(stockManager** head, const char* fileName) {
    FILE* file = fopen(fileName, "r");
    if (!file) {
        printf("Error: Unable to open file for loading.\n");
        return;
    }

    char reference[20], name[20];
    int quantity;
    float sellingPrice, purchasePrice;

    while (fscanf(file, "%s %s %d %f %f", reference, name, &quantity, &purchasePrice, &sellingPrice) == 5) {
        addPart(head, reference, name, quantity, sellingPrice, purchasePrice);
    }
    fclose(file);
    printf("Parts loaded from file.\n");
}

int main() {
    stockManager* head = NULL;
    loadParts(&head, "stockFile.txt");

    int choice;
    do {
        printf("********Stock Manager********\n");
        printf("1 - Add part to the stock\n");
        printf("2 - Remove a part from the stock\n");
        printf("3 - Modify part information\n");
        printf("4 - Display all parts of the stock\n");
        printf("5 - Display benefit of parts in the stock\n");
        printf("6 - Exit\n");
        printf("******************************\n");

        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: {
                char reference[20], name[20];
                int quantity;
                float sellingPrice, purchasePrice;
                printf("Enter the reference of the part: ");
                scanf("%s", reference);
                printf("Enter the name of the part: ");
                scanf("%s", name);
                printf("Enter the quantity of the part: ");
                scanf("%d", &quantity);
                printf("Enter the selling price: ");
                scanf("%f", &sellingPrice);
                printf("Enter the purchase price: ");
                scanf("%f", &purchasePrice);
                addPart(&head, reference, name, quantity, sellingPrice, purchasePrice);
                break;
            }
            case 2: {
                char reference[20];
                printf("Enter the reference of the part to delete: ");
                scanf("%s", reference);
                deletePart(&head, reference);
                break;
            }
            case 3: {
                char reference[20];
                printf("Enter the reference of the part to modify: ");
                scanf("%s", reference);
                stockManager* part = searchParts(head, reference);
                if (part) {
                    int modifyChoice;
                    printf("1 - Add to quantity\n");
                    printf("2 - Subtract from quantity\n");
                    printf("3 - Modify selling price\n");
                    printf("4 - Modify purchase price\n");
                    printf("Enter your choice: ");
                    scanf("%d", &modifyChoice);
                    if (modifyChoice == 1) {
                        int addQuantity;
                        printf("Enter quantity to add: ");
                        scanf("%d", &addQuantity);
                        part->quantity += addQuantity;
                    } else if (modifyChoice == 2) {
                        int subQuantity;
                        printf("Enter quantity to subtract: ");
                        scanf("%d", &subQuantity);
                        if (part->quantity >= subQuantity) {
                            part->quantity -= subQuantity;
                        } else {
                            printf("Not enough quantity in stock.\n");
                        }
                    } else if (modifyChoice == 3) {
                        printf("Enter new selling price: ");
                        scanf("%f", &part->sellingPrice);
                    } else if (modifyChoice == 4) {
                        printf("Enter new purchase price: ");
                        scanf("%f", &part->purchasePrice);
                    } else {
                        printf("Invalid choice.\n");
                    }
                }
                break;
            }
            case 4:
                displayParts(head);
                break;
            case 5: {
                char reference[20];
                printf("Enter the reference of the part to calculate benefit: ");
                scanf("%s", reference);
                stockManager* part = searchParts(head, reference);
                if (part) {
                    float benefit = (part->sellingPrice - part->purchasePrice) * part->quantity;
                    printf("Reference: %s \n Benefit: %.2f DA\n", part->reference, benefit);
                }
                break;
            }
            case 6:
                saveParts(head, "stockFile.txt");
                printf("Exiting program.\n");
                break;
            default:
                printf("Invalid choice. Please enter a number between 1-6.\n");
        }
    } while (choice != 6);

    return 0;
}
