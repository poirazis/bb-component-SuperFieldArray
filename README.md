# Super Field - Array

A dynamic array input component for Budibase applications with add/remove functionality, type validation, and flexible data entry.

## 🚀 Features

### Array Management

- **Dynamic Arrays**: Add and remove array items dynamically
- **Type Support**: String, number, and date array types
- **Field Binding**: Dedicated array field with type safety
- **Default Values**: Pre-populated array data
- **Validation**: Array-specific validation rules

### Data Entry

- **Item Addition**: Easy addition of new array items
- **Item Removal**: Delete individual array items
- **Inline Editing**: Edit items directly in the array
- **Bulk Operations**: Add multiple items at once
- **Order Management**: Reorder array items

### User Experience

- **Visual Interface**: Clear array item display and management
- **Event Handling**: On change events with array context
- **Auto-focus**: Automatic focus for data entry
- **Debounced Input**: Performance optimization for large arrays
- **Help Text**: Guidance for array data requirements

### Advanced Features

- **Type Validation**: Data type validation for array items
- **Conditional Logic**: Dynamic behavior based on array content
- **Button Integration**: Custom action buttons for array operations
- **Icon Support**: Visual indicators for array fields
- **Template Support**: Consistent item formatting

### Styling & Layout

- **Flexible Positioning**: Label placement options
- **Field Modes**: Form input or inline editing
- **Size Configuration**: Adjustable component width
- **Theme Integration**: Consistent with Budibase design

## 📝 Usage Instructions

### Basic Setup

1. Add the Super Field - Array component to your form
2. Bind to an array field in your data source
3. Select the array item type (string, number, date)
4. Configure validation and help text

### Advanced Configuration

- **Item Types**: Choose appropriate data type for array items
- **Validation**: Set array size limits and item validation
- **Buttons**: Configure add/remove action buttons
- **Events**: Attach actions to array changes

### Common Use Cases

- **Tag Management**: User tags or categories
- **Contact Lists**: Email addresses or phone numbers
- **Product Options**: Size, color, or feature selections
- **Task Lists**: To-do items or checklist entries
- **Data Collections**: Multiple values of the same type

## 🔧 Configuration Options

| Setting        | Type    | Description                         |
| -------------- | ------- | ----------------------------------- |
| Field          | Array   | Array field binding                 |
| Type           | Select  | Item data type (string/number/date) |
| Label          | String  | Display label text                  |
| Placeholder    | String  | Item input guidance                 |
| Default Value  | Array   | Pre-populated array data            |
| Help Text      | String  | Help/instruction text               |
| Validation     | Rules   | Array validation (size/item rules)  |
| Autofocus      | Boolean | Auto-focus on load                  |
| Debounced      | Boolean | Enable input debouncing             |
| Disabled       | Boolean | Disable array editing               |
| Read Only      | Boolean | Read-only mode                      |
| Icon           | Icon    | Visual indicator icon               |
| Field Mode     | Select  | Form or inline input style          |
| Label Position | Select  | Label placement                     |
| Size           | Number  | Component width span                |

## 📋 Events

### On Change

Triggered when array items are added, removed, or modified.

**Context:**

- `value`: The current array value
- `field`: The bound field information

## 🎨 Styling

The component supports Budibase's styling system with:

- **Item Layout**: Clean item display and spacing
- **Action Buttons**: Styled add/remove controls
- **Validation States**: Error highlighting for invalid items
- **Responsive**: Mobile-friendly array management

## 🔍 Best Practices

- Choose appropriate item types for data consistency
- Set reasonable limits on array size
- Provide clear labels for array purposes
- Consider mobile users for item management
- Use validation to ensure data quality
- Test array operations with various item counts
